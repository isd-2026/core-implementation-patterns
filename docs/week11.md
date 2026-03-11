# The React Way

## React for dynamic project, allocation list update upon CRUD

React is a modern JavaScript library for building user interfaces designed to improve rendering performance. 
React allows us to create reusable UI components, making code more modular and maintanable. React attaches its own "virtual DOM" to manage rendering and enable components to be imported and composed.
React is a library, not a full framework, meaning that we can integrate it gradually into existing projects without having to refactor everything.

> React may not be the optimal choice for simple static websites and does not support SSR.

## Installation & Setup
> Install [Node.js](https://nodejs.org/en/download)

This also installs the Node Package Manager `npm` by default.

> Create a `vite+react` project

``` bash
npm create vite@latest <project-name> --template react
```

Vite is the development environment and build orchestrator for React applications. It provides a fast development server and optimizes the build process and output structure.
Vite doesn't affect the way your write the core React code.
The above code tells Vite to initialize a React project using its default React template.
The default React template now includes ESLint (for code quality check) as part of its setup (hence you may notice a few additional items in the folder).

> Install necessary packages

The `package.json` file specifies the dependencies and build configurations of the project. This should be included in the root folder of your React project. 
The `"scripts":{...}` section defines commands for building and running the project. For example, `"dev":"vite"` means that running the development script will start the Vite development server. 

Running
``` bash
--prefix ./frontend-react
```
will install all packages listed in `dependencies` and `devDependencies` (packages needed during the development, testing, and building phases) and add them to the `node_modules/` folder. 
When working on a React project with other developers, you share `package.json` (and `package-lock.json`) but not `node_modules/`. 
The `package-lock.json` ensures that everyone installs the exact same package versions, providing consistency across different development environments.

> New frontend folder structure

```
week11/
├── frontend/
│   ├── src/                  # The source folder that contains most of the source code.
|   │   ├── main.jsx          # More details provided below.
│   │   ├── App.jsx           # More details provided below.
│   │   ├── assets/           # Static resources used by the app.
|   |   |   └── style.css     # Style sheet
│   │   └── components/       # More details provided below. 
|   ├── node_modules/         # Auto-generated folder when installing packages with npm.
|   ├── index.html            # More details provided below.
|   ├── package-lock.json     # Ensures consistency across different development environments
│   └── package.json          # Project configuration and dependency list.
└── backend/                  # Same backend as before.
```

- `index.html`
    * The browser’s entry point; static HTML; the browser loads `index.html` before it runs the React code.
    * The Vite development server looks for it in the root folder.
    * Provides the `<div id="root"></div>` where React will render.
    * Points to the React app's entry point `main.jsx` with `<script type="module" src="/src/main.jsx"></script>`.
- `main.jsx`
    * The runtime entry point; bootstraps React.
    * As the browser doesn't understand React JSX or ES module import by itself, you can't directly reference a component file like `<App />` in HTML; after creating a React root on `main.jsx`, you can.
    * `document.getElementById('root')` gets the DOM container from `index.html` and tells React where to render the app.
    * `createRoot(...)` creates a React root.
    * `<App />` renders the top-level component.
    * `StrictMode` wraps the app for development checks (detects unsafe lifecycles, deprecated APIs, etc.).

``` jsx
createRoot(document.getElementById('root')).render(
    <StrictMode>
        <App />
    </StrictMode>,
)
```

- `App.jsx`
    * `function App(){...}` defines a React functional component
    * `return (...)` specifies what JSX is rendered
    * `Router` wraps your app to enable client-side routing
    * `Routes` is the container for all route definitions
    * `<Route path="..." element={<Component />} />` maps a URL path to a child React component

``` jsx
function App() {
    return (
        <Router>
            <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/login" element={<Login />} />
            <Route path="/register" element={<Register />} />
            <Route path="/welcome" element={<Welcome />} />
            <Route path="/register-successful" element={<RegisterSuccessful />} />
            <Route path="/project-status" element={<ProjectStatus />} />
            <Route path="/project-dashboard" element={<ProjectDashboard />} />
            <Route path="/project-form" element={<ProjectForm />} />
            <Route path="/logout" element={<Logout />} />
            </Routes>
        </Router>
    );
}
```

- Individual component, e.g., `./components/Register`, is a reusable piece of UI.

> Run the React frontend from the `week11/` folder

``` bash
npm --prefix ./frontend-react run dev
```

The `npm run dev` command (also called "dev script") starts the project in development mode.

## Re-Write the Frontend in React

Now we take a closer look `./components/Register.jsx`. 

``` jsx
function Register() {
    const [something, setSomething] = React.useState(''); //'' for string default; false for boolean default
    const navigate = useNavigate();

    const handleRegister = () => {...}
    const loadSomething = async () => {
        try {
            const response = await fetch("URL", {...});
            const data = await response.json();
            setSomething(data.something.someattribute);
        } catch (err) {...}
    }
    const handleRegister = async (event) => {
        event.preventDefault();
        ...
    }

    React.useEffect(() => {
        loadSomething();
    }, [dependencies])

    return (/*the returned HTML goes here; wrapped in a a single parent element*/);
};
export default Register;
```

- `function Register() {...};` is a "function components" (also called "functional components") 
    * function components are plain JavaScript functions that accept data as input (called "props") and return React elements that describe what should be rendered.
- `export default Register` 
    * `export` allows this module/file to share something with other files.
    * `default` means this is the primary export of the file.
    * Now you can use `import Register from './Register.jsx';` to import the `Register` component in `App.jsx`.
- React `<Link>`
    * `<Link>` comes from `react-router-dom`.
    * It allows client-side navigation without reloading the page.
    * It supports linking to external URLs and to other React routes.
- React `useEffect`
    * A built-in React hook
        * Only call Hooks at the top-level, not inside loops, conditions or nested functions.
        * Only call Hooks from React function components.
    * A function that allows you to perform side effects, e.g., data fetching, manually modifying the DOM
        + A side effect is any operation that interacts with systems beyond the scope of the current component and its rendering process.
    * Use the `useEffect` Hook when you need to synchronize your component with an external system.
    * Calling `loadSomething` inside `useEffect` prevents it to be called in every render to avoid infinite loops or triggering duplicated API calls.
        + `dependencies` tells React to run this effect again when these values change.
- React `useState`
    * A built-in React hook
    * Adds state variables to functional components and provides a function to update them.
    * Use `useState` when a functional component needs to remember data across re-renders and that data affects the user interface.
    * Prevents triggering duplicated API calls.
- Event handlers to manager user interactions
    * For logic that runs in response to specific user actions, e.g., button click, form input.
- Returned HTML 
    * Convert html (from `register.html`) to jsx (wrapped in a pair of `<>` and `</>` - shorthand for `React.Fragment`) https://transform.tools/html-to-jsx
        + The entire returned JSX in return must be wrapped in a single parent element (commonly a `<div>` or `<>...</>` fragment).
    * Remove everything in <head></head>
        * React components only return body content.
    * React syntax for conditional rendering shorthand `{showLoginLink && <Link to='/Login'>Already have an account? Login here</Link>}`
        + Uses JavaScript logical AND (`&&`) for conditional rendering.
        + React evaluates the expression inside `{}`.
        + If `showLoginLink` is true, the `<Link>` is rendered.
        + If `showLoginLink` is false, React renders nothing.