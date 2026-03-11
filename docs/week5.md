# Flask Backend & Unit Testing

## Setup

> venv

The `venv` module is a built-in Python module (v3.3+) for creating isolated Python environments or "virtual environments". Each virtual environments contains their own independent set of Python packages installed in their site directories.

- start a venv in the `weekx/` folder (we will always run code from this folder) with `$ python -m venv <venv_folder_name>`. By doing so, we put the `venv/` folder in the same directory as the backend project `backend/`.
- (in bash) activate the virtual environment with `$ source <venv_folder_name>/Scripts/activate`
- deactivate the venv with `$ decativate`
- add `**/venv/` to your `.gitignore`

> Running Python scripts as modules

In Python, a relative path is resolved relative to the current working directory (CWD), and you are not restricted to running your script from a particular folder. This can lead to issues when importing or accessing files from other Python scripts if the working directory differs from the script’s location. Therefore, we suggest that you run Python Scripts as modules instead. Below are the steps.

- add a `__init__.py` in the root (Python project) folder (in this case, `backend/`) to enable it to run as module; the file can be empty.
- in the `weekX/` folder, run your script `backend/app.py` as a module with `-m`:`$ python -m backend.app`.

## Flask Backend
> Flask

[Flask](https://flask.palletsprojects.com/en/stable/) is a lightweight, micro web framework for Python. It’s designed for building web applications and APIs quickly.

> Installation

Flask supports Python v3.9+. Activate your venv and install Flask into it with `$ pip install Flask`.

> Minimal backend app with no framework

The code snippet below shows how we create a minimal backend with the `http.server` library only. This enables an API endpoint for a GET request, you need to manually send the response, header

``` python
from http.server import BaseHTTPRequestHandler, HTTPServer
import json

class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        if self.path == "/welcome":
            length = int(self.headers["Content-Length"])
            data = json.loads(self.rfile.read(length))
            name = data.get("name")

            self.send_response(200)
            self.send_header("Content-Type", "application/json")
            self.end_headers()

            self.wfile.write(
                json.dumps({
                    "status": "success",
                    "message": f"Hello, {name}!"
                }).encode()
            )
if __name__ == "__main__":
    HTTPServer(("127.0.0.1", 8080), Handler).serve_forever()
```

where,
- you subclass `BaseHTTPRequestHandler` and define methods like `do_GET`, `do_POST` 
- you handle the request with 
    * `self.path` - the API endpoint
    * `self.headers` - the HTTP headers sent by the client
    * `self.rfile` - the request body
- you construct the response with
    * `self.send_response(200)` - the HTTP status code
    * `self.send_header("Content-Type", "application/json")` and `self.end_headers()` - adds a single HTTP header to the response
    * `self.wfile.write(json.dumps({...}).encode())` - sends the response content to the client
- you run the backend app on the HTTPServer and enable it to listen for requests with `HTTPServer(("127.0.0.1", 8080), Handler).serve_forever()`

> Equivalence in Flask

With Flask, the above code becomes:

``` python
from flask import Flask, request

app = Flask(__name__)

@app.route('/welcome', methods=["POST"])
def welcome():
    data = request.json
    username = data.get("name")
    return {"status":"success", "message": f'Hello, {username}!'}, 200

if __name__ == "__main__":
    app.run(debug=True, host="127.0.0.1", port=8080)
```

Flask simplifies the coding by:
- Creating a Flask application instance `app = Flask(__name__)`
- Using the decorator `@app.route` to map a URL endpoint (`/welcome`) to a Python function (`welcome`). `methods=["POST"]` specifies which HTTP methods this endpoint accepts. It is equivalent to `app.route("/welcome", methods=["POST"])(welcome)`. Note that in Python, a decorator only applies to the function (or class) immediately below it.
- The `request` object, which gives access to the incoming HTTP request data (headers, body, form, JSON); as the request's `Content-Type` is `application/json`, you can use `request.json` to access the payload (body) of the request; eliminating handling of raw bytes or manual json parsing.

> Running the app

- `app.run(debug=True, host="127.0.0.1", port=8080)` 
    * `app.run` starts Flask's built-in development server (usually not used in production)
    * `debug=True` switches on the debug mode which support automatic reloading (on saved changes)
    * the debug mode enables Flask to print detailed information about requests and a detailed traceback in the browser when an error occurs
    * specifies the network interface (`127.0.0.1` or localhost) and port (8080) the server listens on; the full ip address of the server is `http://127.0.0.1:8080/`
- start the server by running `$ python -m backend.app`

> Caveat: You normally create a GET endpoint to retrieve data. However, according to the HTTP specification, GET requests should not include a body, and Flask does not reliably process a payload in a GET request. If you need to send a request with a body, the standard approach is to use a POST endpoint instead.

## Unit Testing

Unit testing is a form of software testing by which isolated source code is tested to validate expected behavior.Unit testing describes tests that are run at the unit-level (e.g., a single function, a single method or a class, a small class or module) to contrast testing at the integration or system level. Unit testing is performed by the developer who writes the code.

> Project bidding [^1]

Project bidding is a simple algorithm to decide if a student can be assigned to a project. The full implementation is provided in `week5/backend/project_bidding.py`. The bidding rules are as follow:

- Each student (user) has 10 bids and can nominate themselves to 1 project
- Students can submit their bids to the three categories: technical, communication, innovation, indicating their ability/confidence in each category
- A project defines the weights for each category
- The `weighted_bids` is calculated as the sum of the weighted bids in all categories
- Each project also defines the maximum number of students, with 6 being the default maximum
- Students are ranked based on their `weighted_bids` and the the top X students are placed on a project where X is equal to or less than the maximum number of students of a project

> Python's `unittest`

The Python `unittest` module is a built-in testing framework that provides tools for creating and running automated unit tests. Below, we explain some of the key syntax. The full implementation is provided in `week5/backend/test_project_bidding.py`.

- Create a test class by extending the `unittest.TestCase` base class, for example, `TestAllocation(unittest.TestCase)`.
    * Each test case is isolated, starting from a known, clean state, and is independent from other tests.
    * `unittest` discovers test methods starting with `test_` and executes them one at a time (execution order not guaranteed).
        + Change file pattern with `python -m unittest discover -p "test*.py"`.
    * For each test, `unittest` calls `setUp()` -> runs the test method -> calls `tearDown()`
- `setUp()` runs before each test case
    * In this example, we create 3 projects and 20 users before each test
- `teardown()` runs after each test case
    * In this example, we remove all the students, projects, registrations and allocations created in each test
- Define test cases, you can name your test cases following the convention `test_<functionality>_<success/failure>`, e.g., `test_registration_success`. The prefix `test_` allows test methods to be discovered by the `unittest` module.
- To check test results, use the following assertions:
    * `assertEqual()` to check whether a value is equal to an expected value.
    * `assertTrue()` to check whether a condition evaluates to `True`.
    * `assertIn()` to check whether a value exists within a collection.
- Automatically run all test scripts in a folder with `python -m unittest discover -s <folder-name>`

> Test coverage

Test coverage may vary depending on the risk, business needs, quality and regulatory requirements. In general, unit tests should cover the core business logic, integration (API) tests should cover key workflows focusing on the most-used paths, end-to-end (e2e) tests should cover core flows. 

In ISD, you are required to demonstrate testing in your project assignment, but the extent of coverage is not assessed.

## Common Vulnerabilities & Mitigations
Cyber security is a complex topic. In ISD, we provide only a very high-level discussion of common website vulnerabilities and their mitigations. In Week 9, when we connect the frontend and backend, we will also move from frontend session storage to backend session storage + frontend cookies.

> Common vulnerabilities mitigated in code (application-level)

There are several additional aspects to consider that can be reflected in the code itself, including common website vulnerabilities and their mitigations:
- Common website vulnerabilities
    * Cross-Site Scripting (XSS): Cross-Site Scripting occurs when an application injects untrusted user input into a web page without proper sanitisation or output escaping, allowing attackers to execute malicious JavaScript in a victim's browser.
    * SQL Injection: The injection of untrusted input into a SQL query in a way that allows it to be interpreted as executable SQL code rather than data. SQL Injection occurs when user input is directly embedded into SQL queries, allowing attackers to manipulate database queries.
- Common mitigations
    * Cross-Site Scripting (XSS):  XSS is commonly mitigated through input validation and output escaping *on the backend*. This includes validating input formats, limiting accepted parameters, setting maximum numbers of parameters, and rejecting unnecessary data in HTTP requests. Modern browsers also support certain HTML5 input validations by default; however, these can sometimes complicate E2E testing.
    * SQL Injection: commonly mitigated by using an Object–Relational Mapping (ORM) framework, such as [SQLAlchemy](https://www.sqlalchemy.org/), which interacts with databases through objects rather than raw SQL, and by using parameterized queries.

> Other vulnerabilities (infrastructure, network, etc)

There are many other vulnerabilities that can occur in the communication channel and hosting environment. For example, a *Man‑in‑the‑Middle (MITM)* attack occurs when an attacker intercepts and potentially alters communication between a client and a server without either party being aware.

In the hosting environment, misconfigurations can also introduce vulnerabilities. For example, exposed database ports or unsecured cloud storage may allow attackers to gain unauthorized access to sensitive data if proper network restrictions and access controls are not enforced.

## Request Handling Efficiency

To ensure request handling efficiency, we may consider the following aspects.

> In code

- Efficient algorithms & database queries.
- Limit individual request cost, e.g., prevent large payloads, apply rate limiting from a user/IP (usually in the backend).
- Use asynchronous processing (`await/async`).
- Reuse database or API connections instead of creating new ones per request.

> Infrastructure

- Select production-grade web servers, such as Nginx, Apache.
- Leverage load-balancers, auto-scaling, firewalls, DDos protection of cloud services.

[^1]: Project bidding logic is not fully incorporated in the sample code
