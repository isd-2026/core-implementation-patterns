# Frontend-Backend Integration

## Implementation
### Sending HTTP requests with JavaScript fetch
The JavaScript `fetch` allows sending HTTP requests. It is used to enable the frontend to send HTTP requests to backend APIs. 

We will demonstrate the use of JavaScript `fetch` through `login.js`.
`login.js` contains the login form that requests the user to enter email and password. 

``` javascript
/*  use data.status to get HTTP response status (success/error)
	use data.message to get the human-readable message from the backend

	display the HTTP response (error/success) in the frontend
*/

fetch("http://127.0.0.1:8080/auth/login", {
	method: "POST",
	headers: {
		"Content-Type": "application/json",
	},
	body: JSON.stringify(credentials),
	credentials: "include" // ensures browser includes cookies
})
	.then((response) => response.json())
	.then((data) => {
		if (data.status === "success") {
			const user = data.user;
			console.log("Login successful:", user.name, user.email);
			window.location.href = "welcome.html";
		} else if (data.message === "Email doesn't exist. Please register.") {
			// display an inline error if the email is not registered
			emailError.textContent = "The email address you entered isn't connected to an account.";
			emailError.style.display = "block";
			document.getElementById("email").focus();
			console.error("Login failed:", data.message);
		} else if (data.message === "Incorrect password.") {
			passwordError.textContent = "The password that you've entered is incorrect.";
			passwordError.style.display = "block";
			document.getElementById("password").focus();
			console.error("Login failed:", data.message);
		}
	})
	// Network / request-level failure (not application error)
	// Examples:
    // - server down
    // - CORS failure
    // - network disconnected
	.catch((error) => console.error("Error:", error));
```

### Frontend Cookie + Backend Session
In Week 4, we used frontend `sessionStorage` to store a user's information after the user has logged in, and pass variables between pages. However, it is not secure because it is fully accessible to JavaScript and offers no built-in access control. Users who control the browser environment can modify `sessionStorage` values using developer tools. Therefore, `sessionStorage` is generally used only for non-sensitive UI data. User details stored in the frontend `sessionStorage` can be stolen upon a successful XSS (cross-site scripting) attack, where the attacker injects JavaScript inside your page that can read from `sessionStorage`.

One common mitigation strategy is to use the pattern *frontend cookie + backend session*. We will walk through how it works using the user login example.

> Frontend Cookie
- A frontend cookie is a small key-value data stored in the user's browser. It is automatically sent to the server with each request.
- You can control the cookie-sending behavior by setting the `credentials` option in the `fetch` request. `credentials: "include"` tells the browser to include cookies (and HTTP auth headers) with all requests, including cross-origin requests. This requires server to allow it via CORS (Cross-Origin Resource Sharing).
- If the cookie is `HttpOnly`, JavaScript cannot read it via `document.cookie` -> inspect with your browser's developer tools (e.g., on Chrome, Inspect -> Application -> Cookies).

> Backend Flask `session`

Storing data in the backend session (physically on the server) is more secure than storing it in the frontend. The steps below explains how Flask backend `session` works with the frontend cookie.
1. Flask session creation
	- when a user logs in, Flask generates a session ID and stores the user info
	- the session data is not sent to the client - only the session ID is
2. Session cookie
	- Flask sends a session cookie to the browser, containing only the session ID
	- the cookie can be configured with flags like `HttpOnly`, `SameSite`, etc
3. Request identification
	- on subsequent request, the browser automatically sends the session cookie
	- Flask reads the session ID from the cookie and retrieves the corresponding session data from the server
	- this allows the server to identify the user and maintain state securely

> Enabling Flask sessions

Add the below settings in `app.py` to enable Flask sessions.

``` python
app.secret_key = "session-secret-key" # required; Flask uses `secret_key` to sign session cookies

app.config.update( 
	SESSION_COOKIE_HTTPONLY=True, # JS cannot read 
	SESSION_COOKIE_SECURE=False, # allow HTTP for local testing 
)
```

The configuration below specifies the frontend origin allowed for HTTP requests. Now you need to access your frontend through the URL "http://127.0.0.1:8000", note that Flask (and the browser's same-origin policy) considers `localhost` and `127.0.0.1` as different origins, and hence if you use the URL "http://localhost:8000" the HTTP request will fail the CORS check. 

``` python
# http://127.0.0.1:8000 is the frontend origin allowed
# r"/*" matches all backend api endpoints
# supports_credentials=True tells Flask to send the proper CORS headers so that browsers will include and accept cookies for cross-origin requests
# altogether: all backend endpoints are configured to allow cross-origin access from http://127.0.0.1:8000.
CORS(app,
	resources={r"/*": {"origins": "http://127.0.0.1:8000"}},
	supports_credentials=True)
```

> Storing user data after the user has logged in

We add the following to the function `login_user` on `auth_controller.py` to store the user data in the backend `session`.

``` python
session.clear() # clears the session for the newly logged in user
session["user_name"] = user.name # stores key-value pair "user_name": user.name in the session object
session["user_email"] = user.email # stores key-value pair "user_email": user.email in the session object
```

> Retrieving user identity from the backend `session`

We have created a `/me` endpoint in `auth.py` which allows HTTP GET requests from the frontend to retrieve the user's identity from the backend `session`.

``` python
@auth_bp.route("/me", methods=["GET"])
def me():
    if "user_name" not in session:
        return jsonify({"status": "error", "message": "Not logged in"}), 401
    return jsonify({
        "status": "success",
        "user": {
            "name": session["user_name"],
            "email": session["user_email"],
        }
    }), 200
```

On the frontend welcome page, once a user is logged in and directed to the welcome page, we confirm the user's identity with the following `fetch` script.

``` javascript
fetch("http://127.0.0.1:8080/auth/me", {
	method: "GET",
	credentials: "include", // include cookies in the request
})
```

> Clear session storage

We also create an api endpoint `/logout` on `auth.py`, which clears the backend `session`.
This is called by the frontend when a user clicks the logout button.

``` python
@auth_bp.route("/logout", methods=["GET"])
def logout():
    session.clear()
    return jsonify({"status": "success", "message": "Logged out successfully"}), 200
```

### DOM Manipulation With JavaScript
On the welcome page, we display a logout button for logged-in users and a message prompting users to log in if they are not authenticated. This requires different UI for different users. We use JavaScript below to insert the logout button so that it is only rendered for logged-in users.
- `document.createElement` creates the `button` element
- `appendChild` appends the logout button to the `mainContainer` element

``` javascript
const logoutBtn = document.createElement("button");
logoutBtn.id = "logoutBtn";
logoutBtn.textContent = "Logout";
mainContainer.appendChild(logoutBtn);
```

## UML Sequence Diagram
Now let's visualise how the entire system connects thorugh a sequence diagram. You can refer to the example [Login Sequence Diagram](https://miro.com/app/board/uXjVGCAiuyA=/?share_link_id=10398528666) on Miro.

A sequence diagram is an interaction diagram that emphasizes the time-ordering of messages. It can be used to illustrate particular senarios involving the user interacting with the system. It should include:
- External actor 
- Lifeline of all participants
- Messages
- Sequence fragments (also called combined fragments, including loops, branches, alternatives)

For more details on notations, refer to the [Visual Paradiagm Guide](https://www.visual-paradigm.com/learning/handbooks/software-design-handbook/sequence-diagram.jsp).

## Summary of Major Code Changes
- `logout.html`
- `register_successful.html`
	* Confirms registration status after successful student registration
- `welcome.html`
	* Student page after successful login.
- `app.py`

## Extensions 
> Server-Side Rendering (SSR)
Server-side rendering ([SSR](https://developer.mozilla.org/en-US/docs/Glossary/SSR)) is the process of generating the HTML of a web page on the server instead of in the browser. 
The fully rendered page is then sent to the client, which reduces the amount of client-side JavaScript execution. 
Frameworks such as [Next.js](https://nextjs.org/docs/pages/building-your-application/rendering/server-side-rendering) supports built-in SSR. SEO and performance are also improved because crawlers receive fully rendered HTML.
> Secure webside design considerations
- HTTPS
- Input validation & output Encoding
- Secure cookies
- Many more ...