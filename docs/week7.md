# Connecting Backend to the Database

> Model of Model-View-Controller (MVC)

MVC is an architectural pattern commonly used to enable modular designs and enforce separation of concerns based on the responsibility of each component. 
- View is the interface that presents information to and accepts it from the user.
- Model represents application data, enforces business rules, and manages database access.
- Controller orchestrates requests from the view to the Model by interpreting the requests and invoking appropriate logics on the Model; it also decides how responses from the Model should be rendered on the View.

In ISD, we use a hybrid frontend-backend MVC, with the frontend serving as the View. With a pure server-side rendering (SSR) framework, the entire MVC pattern can be embedded in the backend.

> Data Access layer (DAL)
Here we introduce a manual DAL as part of the Model of our MVC, implemented in `db_crud.py` in the `/backend/model/` folder. The DAL handles database connections, queries and commands, and maps database records to application data structures. 

The `add_user()` function below defines how to insert a new user record in the database. We use the `get_connection()` function defined in previous weeks. Once we create a `cursor()` instance, we execute the SQL query to insert a user record to the database. 

An `sqlite3.IntegrityError` occurs when a database operation violates a constraint defined in the SQLite table schema. The schema is defined in `db_init.py` which requires the email field to be UNIQUE. Hence if a user tries to register with an already registered email, an error will be raised. 

Sometimes the errors returned directly from the db can be hard to interpret; therefore, it is helpful to return meaningful error messages when defining route functions to improve usability and assit with debugging.

```
def add_user(name, email, password,gender, favcol):
    """Insert a new user into the database."""
    conn = get_connection()
    cursor = conn.cursor()
    try:
        cursor.execute("""
            INSERT INTO users (name, email, password, gender, favcol)
            VALUES (?, ?, ?, ?, ?)
        """, (name, email, password, gender, favcol))
        conn.commit()
    except sqlite3.IntegrityError as e:
        raise ValueError(f"Email '{email}' already exists.") from e
    finally:
        conn.close()
```

Note that because we still pass raw SQL queries directly, this manual DAL is not considered an ORM.

## Connecting Backend to the Database

Routes accept HTTP requests and query the database through methods defined in the Model. A route module (i.e., Blueprint) can define multiple API endpoints, each of which specifies the path, HTTP method, request parsing, validation, business logic calls, and response construction.

> Flask Blueprints

Flask Blueprints help organize routes into modular, reusable components. A Blueprint represents a group of related routes. You can develop, test, and maintain different parts of your app separately, and then register them with your main application.

In the `/backend/routes/` folder we created two routes modules. 
- `auth.py` for managing authentication, e.g., login, registration 
- `users.py` for managing users, which is primarily accessed by the system admin to manage user information. In `app.py`, we import the two Blueprints and register them with the application as follows.

```
app.register_blueprint(users_bp,url_prefix = "/users")
app.register_blueprint(auth_bp,url_prefix = "/auth")
```

> Endpoint function

An endpoint defines the access point (a URL + an HTTP method); the endpoint function defines the behavior. Let's look at the implementation of API endpoint: `http://127.0.0.1/auth/login` in `auth.py`.

``` python
auth_bp = Blueprint("auth",__name__)

@auth_bp.route('/login',methods=["POST"])
def login():
    data = request.json
    email = data.get("email")
    password = data.get("password")

    # server-side validation
    if not email or not password:
        return jsonify({"status":"error", "message":"Email and password required"}), 400

    user_row = get_user_by_email(email) # returns a tuple not an object
    if user_row is None:
        return jsonify({"status":"error", "message": "Email doesn't exist. Please register."}), 404
    
    if user_row[3] != password:
        return jsonify({"status":"error", "message": "Incorrect password."}), 401
    
    # session["user_email"] = user_row[2]
    return jsonify({
        "status": "success",
        "message": "Login successful",
        "user": {
            "name": user_row[1],
            "email": user_row[2],
            "gender": user_row[4],
            "favcol": user_row[5]
        }
    }), 200
```

- `auth_bp = Blueprint("auth",__name__)` creates a Blueprint object
- the `@auth_bp` decorator tells Flask to register this function as a route under the auth Blueprint.
- the `@auth_bp` decorator is followed by an *endpoint function* `login()`. When a matching request arrives, Flask invokes this function. Each endpoint is mapped to exactly one endpoint function. 
- this API endpoint accepts `POST` http requests; 
- it retrieves the email and password fields from the request's payload
- it validates (server-side validation) if the email or password is empty
- it uses `get_user_by_email(email)` to fetch the user record (if found) as a `sqlite3.row` object and checks if the password entered matches with the database record.
- the return values are in a unified format (across all endpoints): `{"status":"error" | "success", "message":<meaningful message>}, <status-code>`. 

> API Key

All users including students and the admin need to log in through the authentication process. However, user management is primarily relevant to the admin only, and one user should not have access to other users' information.

A common way to restrict access is through an API Key, which is only known to the user who should have access to the particular data. 

We define a `require_api_key` decorator as follow:
```
ADMIN_API_KEY = "admin-secret-key"

def require_api_key(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        api_key = request.headers.get("x-api-key")
        if api_key != ADMIN_API_KEY:
            return jsonify({"status": "error", "message": "Invalid or missing API key"}), 401
        return f(*args, **kwargs)
    return decorated
```

The line `api_key = request.headers.get("x-api-key")` requires that the request to this API endpoint includes a header `x-api-key`. We then check if it is equal to the `ADMIN_API_KEY`, which is hardcoded in the backend in this case. For real-world security, the API key would typically be generated and managed dynamically. 

To apply a decorator, you place it directly above the endpoint function that you want to restrict access to.

## Unit Testing for DAL

We create a test class TestDBCRUD in `backend/tests/test_db.py`. In `setUp()` we drop the existing `users` table (this must never be done against a production database) and create a new one to ensure a clean test state, and in `tearDown()` we drop the users table created.

Note that `conn.commit()` is called in all functions in `db_crud.py`. As a result, when these tests are executed, database state changes are persisted to the database until they are explicitly reset (for example, in `setUp()` or `tearDown()`).

Run the test with `python -m unittest backend.tests.test_db`.