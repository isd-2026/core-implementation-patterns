# Extending Software Features

Now let's extend the functionality of the software. We also introduce a new role — the admin, who has different access from a student. We will enable the admin to manage projects and allow students to self-assign to projects.

## Handling Asynchronous Updates

### Asynchronous Loading
Creating a new or modifying a project updates the project list. To ensure the frontend shows the user the up-to-date list of projects, we define `loadProjects` as follows. In `project-dashboard.js`, we call `loadProjects` on the initial page load and after a project is removed.

``` javascript
async function loadProjects() {
    try {
        const response = await fetch("http://127.0.0.1:8080/projects/list");
        const data = await response.json();

        if (data.status !== "success") return;

        allProjects = data.projects;
        renderProjects(allProjects);
    } catch (err) {
        console.error("Error loading projects:", err);
    }
}
```

If you want continuous updates of projects, you can configure `loadProjects` to be called at a specific interval (e.g., every 5000 ms) with
``` javascript
setInterval(loadProjects, 5000);
```

### Avoid Race Conditions
Asynchronous operations (Promises, `setTimeout`, `fetch`, event listeners) can complete in an unpredictable order, resulting in race conditions. For example, we want to make sure only the admin can create a project (only the admin can access the create-project page). If we check the user's role and render the UI in the following manner, we may have a race condition, as `fetch` is async and the role check is not guaranteed to complete before the UI is rendered.

``` javascript
let role = null;

fetch("http://127.0.0.1:8080/auth/me", {
    credentials: "include"
})
.then(response => response.json())
.then(data => {
    if (data.status === "success") {
        role = data.user.name;
    } else {
        console.error("Failed to fetch role:", data.message);
    }
})

if (role === "admin") {...}
```

Therefore we should wait for the role check to complete before rendering the UI:

``` javascript
let role = null;

fetch("http://127.0.0.1:8080/auth/me", {
    credentials: "include"
})
.then(response => response.json())
.then(data => {
    if (data.status === "success") {
        role = data.user.name;
        if (role === "admin") {...}
    } else {
        console.error("Failed to fetch role:", data.message);
    }
})
```

As our codebase becomes more complex, we refactor the frontend into a more modular design by defining handler functions. `fetchUserRole` is an async function to get the user's role. `handleCreateProject` handles project creation when the create button is clicked.

``` javascript
let role = null; 

async function fetchUserRole() {
    try {
        const response = await fetch("http://127.0.0.1:8080/auth/me", {
            credentials: "include"
        });
        const data = await response.json();
        role = data.user.name;
    } catch (err) {
        console.error("Failed to fetch role:", err);
    }
}

async function handleCreateProject() {
    formError.textContent = "";

    const projectName = projectNameInput.value.trim();
    const clientName = clientNameInput.value.trim();

    try {
        fetch("http://127.0.0.1:8080/projects/create", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                "Authorization": "admin-secret-key"
            },
            body: JSON.stringify({
                name: projectName,
                client: clientName,
            }),
        })
            .then((response) => response.json())
            .then((data) => {
                if (data.status !== "success") {
                    alert("Error creating project: " + data.message);
                    return;
                }
            });

        alert("Project created successfully!");
        projectNameInput.value = "";
        clientNameInput.value = "";
        projectNewNameInput.value = "";
        clientNewNameInput.value = "";

    } catch (err) {
        console.error("Error creating project:", err);
    }
}

function renderCreateMode() {
    projectNameInput.style.display = "block";
    projectNameTr.style.display = "table-row";
    clientNameInput.style.display = "block";
    clientNameTr.style.display = "table-row";

    projectNewNameInput.style.display = "none";
    projectNewNameTr.style.display = "none";
    clientNewNameInput.style.display = "none";
    clientNewNameTr.style.display = "none";

    createBtn.style.display = "inline-block";
    updateBtn.style.display = "none";

    createBtn.addEventListener("click", handleCreateProject);
}
```

Then we use a selection statement to render UI depending on the user's role.

``` javascript
await fetchUserRole();

if (role === "admin") {
    renderCreateMode();
} else {
    renderSomethingElse();
}
```

## E2E Testing

End-to-end (E2E) testing is performed to verify the overall behavior of a full-stack application. It often focuses on the frontend, simulating user interactions through the browser.

*Selenium* is a widely used framework for E2E testing and is considered an industry standard. It uses drivers to open browsers and perform actions such as entering inputs, clicking buttons, and navigating pages, etc, effectively simulating real user behavior. Selenium tests require the specified browser to be installed; and the frontend server to be running.

At this stage, our frontend and backend are not yet integrated, but we can begin implementing frontend E2E tests to validate its behavior independently. Selenium available in Python, JavaScript, etc. In ISD, we only study the Python version of Selenium. 

## Python Selenium

Selenium interacts with HTML elements, and one method for selecting elements is to get elements by their ID using `find_element(By.ID, <element-id>)`. We will run through a frontend testing example (full implementation available in `test_register_e2e.py`) to test the behavior of our `register.html`. You will need to import the following packages.

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoAlertPresentException
from selenium.webdriver.chrome.options import Options
```

Then you can use the following syntax to perform user actions and access HTML elements.
- `driver = webdriver.Chrome(options=chrome_options)` creates a new Selenium WebDriver instance for the Google Chrome browser. `options=chrome_options` signals the options are set to defaults, meaning that nothing is changed from Chrome's normal behavior.
- `driver.get("http://localhost:8000/register.html")` instructs the driver to navigate to a specific URL.
- `self.form = self.driver.find_element(By.ID, "register-form")` locates an HTML element with the id `register-form` on the currently loaded page. 

Run the test script with command `python -m unittest frontend/tests/e2e/test_login_e2e.py`.

## Managing Tests

We use Python's `unittest` module to organize and run E2E tests similar to what we did for unit tests. While more specialised testing frameworks exist, `unittest` is sufficient functionality our purpose.
We use one test class for each HTML page. 
First we create a test class `TestRegister`. In `setUp()` we load the `register.html` page and locate the registration form. In `tearDown()`, we quit the driver instance and call `super.tearDown()` to cleanup. 

Given a test case `test_name_empty`,
```
def test_name_empty(self):
    # check the tos box if not already checked
    tos_checkbox = self.form.find_element(By.ID, "tos-checkbox")
    if not tos_checkbox.is_selected():
        tos_checkbox.click()

    # no fields are filled, submit the form
    self.form.submit()

    try:
        WebDriverWait(self.driver, 10).until(EC.alert_is_present())
        alert = self.driver.switch_to.alert
        alert_text = alert.text

        if alert_text != "Please enter your name.":
            raise Exception(f'Expected "Please enter your name.", but got "{alert_text}"')

        alert.dismiss()
        print("Passed: shows alert if name is empty")
    except (TimeoutException, NoAlertPresentException):
        raise Exception("Expected alert did not appear")
```

- `tos_checkbox.click()` simulates a mouse click on the `tos-checkbox` element.
- `self.form.submit()` submits the form.
- `WebDriverWait(self.driver, 10).until(EC.alert_is_present())` pauses the test execution for up to 10s, until an alert dialog appears.  
- `alert = self.driver.switch_to.alert` switch the driver's focus from the main page to the alert dialog, and returns an `Alert` object, allowing Selenium to read the alert text and accept or dismiss that alert.

> Caveat: HTML5 form validation is performed by modern browsers and may cause E2E tests to fail in certain cases. For example, our `<input>` field password is of type password. Chrome automatically displays a browser-generated alert when the field is empty, which differs from our custom alert "Please enter your password.". If the test checks the alert text using exact matching, it will fail.

## Summary of Major Code Changes
- `auth_controller.py`, `auth.py`
    * Enables backend session storage for a logged-in user's identity
    * Enables retrival of user's identity from backend session
    * Clears backend session when a user logs out
- `login.js`, `login.html`
	* Change the password input type to `text` to demonstrate E2E testing (bypassing browsers HTML5 validations)
    * Enables role selection upon logging in
    * Enables backend session storage for a logged-in user's identity 
- `logout.js`
    * Clears backend session storage through JavaScript `fetch`
- `test_login_e2e.py`, `test_register_e2e.py`
    * E2E testing scripts with limited test cases.
- `welcome.html`, `welcome.js`
    * Show a link to the project dashboard if no project is found
- `project-form.html`, `project-form.js`
    * Enables admin to create and update a project.
- `project-dashboard.html`, `project-dashboard.js`
    * Shared by students and admin. Students can view (search) all projects; admin can view (search), create, edit, delete.
    * Delete a project removes all its allocations.
- `project-status.html`, `project-status.js`: 
    * Shared by students and admin. Student can self-assign to one project; admin can assign a student to and remove a student from a project.