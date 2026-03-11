# Frontend Basics

## Static Webpages with HTML

In most web servers, if you request a directory path (e.g., `http://localhost:8080/`) rather than a specific file, the server will automatically look for a default file, usually `index.html`.

Below is a code snippet for a very basic `index.html`.

``` html
<!doctype html>
<html>
  <head>
    <title>Home</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div class="content-container">
      <h1>Project Management System</h1>
      <nav>
        <a href="index.html">Home</a>
        <a href="register.html">Register</a>
      </nav>
      <img
        src="sunflower.jfif"
        alt="A beautiful sunflower"
        title="Click to download this sunflower"
        class="main-item"
      />
    </div>
  </body>
</html>
```

HTML files are composed of HTML elements, which define the structure and content of a web page. HTML elements are created using tags. Below is an explanation of the elements used in this snippet. A complete reference for HTML elements can be found at [HTML Elements Reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements).

- `<!doctype html>` declares the document type and version of HTML. It ensures the browser renders the page in standards mode.
- `<html>` is the root element of the HTML document. All other elements are nested inside it.
- `<head>` and `<body>`
    *  `<head>` contains metadata about the document, such as the page title, character encoding, linked stylesheets, and scripts.
    *  `<body>` contains all the content that is displayed to the user, including text, images, links, and other HTML elements.
- `<title>` specifies the title of the document, which appears in the browser tab or window title bar.
- `<link rel="stylesheet" href="style.css" />` links an external CSS file to style the HTML page. The `rel="stylesheet"` attribute specifies the relationship, and `href="style.css"` points to the CSS file.
- `<div class="content-container">` is a generic container element used to group and style content. The `class` attribute allows this div to be targeted with a class selector in CSS `.content-container`.
- `<h1>` is the main heading of the page.
- `<nav>` defines a navigation section with links to other pages of the website.
- `<img src="..." alt="..." title="..." class="..."/>` inserts an image. `src` specifies the path to the image file,[^1] and when downloading this file the browser uses the `src` filename as the downloaded name. `alt` provides alternative text for screen readers and when the image cannot be displayed. `title` shows a tooltip when the user hovers over the image.

## File Organization & Run Your Frontend
Put all your HTML files in the `frontend/` folder. In future weeks, we will assume you run the code from the `weekx/` folder. 

We will use `http.server` which is Python's built-in web server to server your files. Run the command `$ python -m http.server 8000 --directory frontend/` to start the server. Here `-m http.server` runs Python's HTTP server as a module. `8000` specifies the port number. `--direcotry frontend/` tells the server to serve files from the `frontend/` folder.

After starting the server, open your browser and enter the url `http://127.0.0.1:8000` to view the website served locally.

## JavaSctipt for Element Behaviours

JavaScript is used to control the behavior of HTML elements, such as what happens when a button is clicked, when a user enters input into a form, or when a page is dynamically updated. Let’s walk through an example of a form submission.

> User Registration Example

We want to create a page for a user to register. The file `register.html` contains a registration form (of type `<form>`) with several `<input>` fields that are monitored. When the form is submitted (i.e., the `Register <button>` is clicked), the user is taken to a `welcome.html` page. The values of the input fields, such as `name` and `email`, can either be passed through the URL or stored in `sessionStorage`.

You can include the JavaScript file `register.js` and refer to it in `register.html` with `<script src="register.js"></script>`. Place it at the end of the `<body>` so that the DOM elements are loaded before the script executes. The following code snippet shows a simple `register.js` where

- `document.addEventListener("DOMContentLoaded", function () {});` adds an event listener to the HTML document. The `DOMContentLoaded` event fires when the DOM (Document Object Model) has been fully loaded and parsed. 
- the callback function `function() {}` (unnamed in this case) runs when the DOM content is loaded. Callback functions are an essential concept in JavaScript for handling asynchronous events.
- `const form = document.getElementById("register-form");` finds the HTML element with id `register-form` and stores it in the form variable. Similarly, `const name = document.getElementById("name").value` gets the value the user typed into the input with id `name`.
- `form.addEventListener("submit", function (event) { ... });` adds a listener for the `submit` event of the form.
  * `function(event) {}` is an unnamed callback.
  * The browser automatically calls your callback when the event happens.
  * It passes the Event object as the first argument.
  * Your event parameter receives this object.
  * Inside the function, you can use event to access all details about the event.
  * `event.preventDefault();` prevents the form from performing its default action, which is to send data to the server and reload the page.

``` js
document.addEventListener("DOMContentLoaded", function () {
  const form = document.getElementById("register-form");

  form.addEventListener("submit", function (event) {
    event.preventDefault(); // stop form from automatically submitting, required

    const name = document.getElementById("name").value;
    const email = document.getElementById("email").value;

    // do something

  });
});
```

*Client-side validation* is primarily about improving the user experience and reducing unnecessary server load, and is not a security measure. On `register.html`, we add the `required` attribute to the checkbox element `<input type="checkbox" id="tosCheckbox" name="option1" value="yes" required/>`. This allows the browser to check whether the checkbox has been ticked before the user submits the form and provide immediate feedback if it hasn't. Other checks you may perform with client-side validation include type or pattern checks for input fields (example available from Week 9). Security-critical validation should be performed on the server side (example available from Week 7).

> Dynamic Data Passing via URL

*query parameters* are a common method to pass dynamic data from the client (browser) to the server via the URL. You can construct a URL like `http://example.com/page?name=Jane&age=55` to send "Jane" as the name and 55 as the age. All query parameters are sent as strings in the URL, so their types need to be handled by the server. Continue with `register.js`, we've added the logic to redirect user to the welcome page with the user name shown in the URL, where

- `if (name.trim() === "" || email.trim() === "")` checks if either the name or email field is empty. `name.trim` removes leading and trailing spaces from the input.
- `alert("Please enter your name and email.");` shows a popup message prompting the user to fill in the fields on the browser.
- `window.location.href` changes the current page URL, effectively redirecting the browser to another page.
- `welcome.html?name=${encodeURIComponent(name)}&email=${encodeURIComponent(email)}` uses *template literal* to embed variables into strings, `encodeURIComponent(name)` ensures special characters like spaces, &, ? are URL-safe.

``` js
document.addEventListener("DOMContentLoaded", function () {
  const form = document.getElementById("register-form");

  form.addEventListener("submit", function (event) {
    event.preventDefault(); // stop form from automatically submitting, required

    const name = document.getElementById("name").value;
    const email = document.getElementById("email").value;

    if (name.trim() === "" || email.trim() === "") {
      alert("Please enter your name and email.");
    } else {
      // Redirect to welcome page with name in URL
      window.location.href = `welcome.html?name=${encodeURIComponent(name)}&email=${encodeURIComponent(email)}`;
    }
  });
});
```

To retrive the variables on the welcome page, we create a `welcome.js`, and use `URLSearchParams(window.location.search)`: 

``` js
document.addEventListener("DOMContentLoaded", function () {
  const welcomeMessage = document.getElementById("welcome-message");
  const logoutButton = document.getElementById("logoutBtn");

  const params = new URLSearchParams(window.location.search);
  const name = params.get("name");
  const email = params.get("email");

  if (welcomeMessage) {
    if (name) {
      welcomeMessage.textContent = `Welcome, ${name}! \n Your email address is ${email}.`;
    } else {
      welcomeMessage.textContent = "Please register!";
    }
  }
});
```

Query parameters, however, expose all data in the URL (visible to users and browser history) and are unsuitable for sensitive or complex data.

>Using `sessionStorage` 

`sessionStorage` allows you to store data for the current browser session. The data is cleared when the tab or browser is closed. In the code below, `sessionStorage.setItem("user", JSON.stringify(user))` stores the user as a json in the `sessionStorage`, which can be retrieved on `welcome.js` with `const user = JSON.parse(sessionStorage.getItem("user"));`. You should clear the `sessionStorage` with `sessionStorage.clear();` to remove all data stored in the browser session, e.g., when a user logs out. 

Example `register.js`
``` js
document.addEventListener("DOMContentLoaded", function () {
  const form = document.getElementById("register-form");

  form.addEventListener("submit", function (event) {
    event.preventDefault(); // stop form from automatically submitting, required

    const name = document.getElementById("name").value;
    const email = document.getElementById("email").value;

    const user = {"name":name, "email":email}
    sessionStorage.setItem("user", JSON.stringify(user))
    window.location.href = "welcome.html"
  });
});
```

Example `welcome.js`
``` js
document.addEventListener("DOMContentLoaded", function () {
  const welcomeMessage = document.getElementById("welcome-message");
  const logoutButton = document.getElementById("logoutBtn");

  const user = JSON.parse(sessionStorage.getItem("user"));

  if (user) {
    welcomeMessage.textContent = `Welcome, ${user.name}! \n Your email address is ${user.email}.`;
  } else {
    welcomeMessage.textContent = "Please register!";
  }

  logoutButton.addEventListener("click", function () {
    sessionStorage.clear();
    window.location.href = "logout.html"
  })
});
```

> `localStorage`

Is persistent, which means the data remains even if you:
- Close the browser
- Restart the computer
- Refresh or navigate away from the page

## Managing Asynchronous Operations in JavaScript: Callbacks, Promises, and Async/Await [^2]
JavaScript is single-threaded meaning it can only do one thing at a time. If you try to run long operations (like network requests, file I/O, or timers) synchronously, it will block everything else (e.g., UI rendering), including updating the UI or responding to user actions. 

*Asynchronous functions* allow other code to continue executing while a task is waiting, which means the order of completion is not guaranteed (leading to potential *Race Conditions* - see below). Because of this, you often use callbacks (or Promises / async-await) to ensure code runs after the asynchronous task finishes. (See below)

### Callbacks (Callback Functions)
In JavaScript, a callback is a function that is passed as an argument to another function and is executed later, usually after some operation completes. Callbacks define what should run after an operation completes. Callbacks themselves are normal (synchronous) functions. Callbacks are very common in async operations like timers, events, and I/O. 

``` js
function createAudioFileAsync(audioSettings, callback) {
  setTimeout(function() {
    callback(result);
  }, 1000);
}
```

In the example above,
- `callback` is a function, which `createAudioFileAsync` will call later with the `result`.
- `setTimeout` is a built-in *asynchronous function*; the first argument `function() { ... }` calls the `callback` function (after the specified timeout); the second argument is the delay/timeout in milliseconds.
  * Suitable for operations such as HTTP reqeuests, accessting a database, getting user input, etc.

### Promises
A Promise is an object representing the eventual completion or failure of an asynchronous operation. Asynchronous functions often return a Promise instead of returning the value directly. 

Instead of passing callbacks into a function, you can attach callbacks to a Promise. The *register callbacks* `.then`, `.catch`, `.finally` are methods used on a Promise object that run when the Promise settles (either *fulfilled (resolved)* or *rejected*)
- `.then(onFulfilled, onRejected)` registers callbacks for success and failure. 

``` js
promise
  .then(
    value => console.log("Success:", value), // onFulfilled
    error => console.error("Error:", error)  // onRejected
  );
```
- `.catch(onRejected)` registers a callback for rejection only

``` js
promise
  .catch(error => console.error("Caught error:", error));
```

- `.finally(onFinally)` registers a callback that runs regardless of success or failure (doesn't receive the resolved value or the rejection reason)


> With callback-style syntax, we define `createAudioFileAsync`, with a `successCallback` and a `failureCallback`

``` js
function successCallback(result) {
  console.log(`Audio file ready at URL: ${result}`);
}

function failureCallback(error) {
  console.error(`Error generating audio file: ${error}`);
}

createAudioFileAsync(audioSettings, successCallback, failureCallback);
```

> Using Promises, the above example becomes

``` js
createAudioFileAsync(audioSettings).then(successCallback, failureCallback);
```

### Chaining
A common need is to execute two or more asynchronous operations back to back, where each subsequent operation starts when the previous operation succeeds, with the result from the previous step. Doing several asynchronous operations in a row in the callback-style syntax would lead to the "classic callback hell":

> Classic callback hell

``` js
doSomething(function (result) {
  doSomethingElse(result, function (newResult) {
    doThirdThing(newResult, function (finalResult) {
      console.log(`Got the final result: ${finalResult}`);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

In the above code,
- `doSomething` is an asynchronous function, it takes a success callback (`function(result) { ... }`) and a failure callback (`failureCallback`)
- `function(result)` is an anonymous, normal (synchronous) function; it receives the output `result` of `doSomething` when it executes successfully
- `doSomethingElse` is a step inside `function(result)`; 
- `doSomethingElse` is an asynchronous function with a success callback (`function(newResult) { ... }`) and a failure callback

> With Promises, the above code becomes:

``` js
doSomething()
  .then(result => doSomethingElse()) // doSomethingElse is independent from result
  .then(newResult => doThirdThing(newResult)) // doThirdThing depends on newResult
  .then(finalResult => console.log(finalResult))
  .catch(failureCallback);
```

where,
- Asynchronous functions `doSomething()`, `doSomethingElse()`, `doThirdThing()` each returns a Promise (object)
- `.then` waits for the Promise to resolve and passes the resolved value to the next `.then()`

> `async/await` syntax
`async/await` is built on top of Promises (for clarity). The `async` function declaration creates a binding of a new `async` function to a given name. The `await` keyword is permitted within the function body, enabling asynchronous, promise-based behavior to be written in a cleaner style and avoiding the need to explicitly configure promise chains. The equivalent `async/await` version of the above code is:

``` js
async function runTasks() {
  try {
    const result = await doSomething();
    const newResult = await doSomethingElse(result);
    const finalResult = await doThirdThing(newResult);
    console.log(`Got the final result: ${finalResult}`);
  } catch (err) {
    failureCallback(err);
  }
}
```

### Avoid Race Conditions
A race condition happens when two or more asynchronous operations compete, and the program behavior depends on the order they finish.

> Common Causes in JS
- Concurrent async calls modifying shared state
- Event handlers firing in unpredictable order
- DOM updates depending on async results
- Timers (setTimeout, setInterval) conflicting with async code

> Strategies to Avoid Race Conditions
- Use await to sequence async operations
- Chain Promises instead of firing them blindly

## Styling with Plain CSS
Include your CSS style sheet in the `<head>` element on your HTML files as follow.

``` html
<head>
  <title>Account Page</title>
  <link rel="stylesheet" href="style.css" />
</head>
```

> Element Selector

An *element selector* styles all elements of a given type. E.g., the following code snippet applies the defined style to all `<h1>` elements. More specifically, `color: #0000FF;` and `font-size: 40px;` sets the color and font size. `text-align: center;` centers the text horizontally within its parent container. `margin-bottom: 20px; ` adds 20 pixels of space below the heading.

``` css
h1 {
  color: #0000FF;
  font-size: 40px;
  text-align: center;
  margin-bottom: 20px;
}
```

The following code snippet uses *descendant styling* to target all `<a>` elements (for links) that are inside a `<nav>` element (a container for navigation links).

``` css
nav a {
  display: inline-block;
  margin: 0 10px;
  text-decoration: none;
  color: #007bff;
}
```

> Class Selector

A *class selector* styles all elements with a specific class. Classes can be reused on the same page (multiple elements can share the same class) and across different pages.

``` css
.content-container {
  background: white;
  padding: 30px 40px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  width: 600px;
  text-align: center;
}
```

In the example above, `.content-container` targets any HTML element with the class `content-container`. `background: white;` sets the background color of the container to white. `padding: 30px 40px;` adds inner space between the content and the container border. `border-radius: 8px;` rounds the corners of the container by 8 pixels. `box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);` creates a shadow effect behind the container.

> ID Selector

An *id selector* styles one unique element.
  * IDs should be unique per page.
  * IDs can be reused across different pages.

``` css
.content-container #project-search {
  margin-bottom: 12px;
  padding: 6px;
  width: 100%;
}
```

The code snippet above shows how we style an element with the id `project-search` inside any element with the class `content-container`. `margin-bottom: 12px;` adds 12 pixels of space below the element, separating it from whatever comes next. `padding: 6px;` adds 6 pixels of space inside the element, between its content (e.g., text) and its border. `width: 100%;` makes the element take up the full width of its parent container (in this case, the `content-container`).

> Remarks

Styling can be complex and tricky as CSS styles can "override" each other because of specificity, source order, and importance. In general,
- More specific selectors win over less specific ones.
- When two rules have the same specificity, the last one in the CSS wins.
- A rule with `!important` will override almost everything else, unless another rule with higher specificity also has !important. e.g., `color: blue !important;`.
- Some properties come from parent elements (inheritance) unless explicitly overridden

Please refer to the [CSS: Cascading Style Sheets](https://developer.mozilla.org/en-US/docs/Web/CSS) for more information. More recent frameworks, such as [Tailwind CSS](https://tailwindcss.com/), are also widely used and may simplify styling. You may additionally use design tools such as [Figma](https://www.figma.com/), which allow you to lay out interfaces visually and generate corresponding HTML and CSS.

In ISD, we do not require sophisticated styling, as long as your website maintains a consistent look across different pages.

[^1] You can use relative paths (paths relative to the location of the HTML file), or absolute paths (root-relative path, e.g., `<img src="/images/sunflower.jfif" />` where `/` means start from the root of the website, not the HTML file location), or full URL (e.g., `<img src="https://example.com/images/sunflower.jfif" />
` for broswer to fetch the image from the external server).
[^2] https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS