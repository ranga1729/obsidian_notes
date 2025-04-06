Here are some **intern-level React.js interview questions** along with their answers:
### **Basic React Questions**

#### 1. What is React?
**Answer:**  
React is a **JavaScript library** for building user interfaces. It is component-based, declarative, and allows developers to build reusable UI components. React is mainly used for developing **single-page applications (SPAs)**.
#### 2. What are the key features of React?
**Answer:**
- **JSX (JavaScript XML):** Syntax extension for writing UI elements in JavaScript.
- **Virtual DOM:** Efficiently updates the UI by comparing changes with a lightweight copy of the real DOM.
- **Component-Based Architecture:** UI is broken down into small, reusable components.
- **Unidirectional Data Flow:** Data flows in a single direction, making debugging easier.
- **Hooks:** Allows functional components to manage state and side effects.
#### 3. What is JSX?
**Answer:**  
JSX stands for **JavaScript XML**. It is a syntax extension that lets us write HTML-like code inside JavaScript. It makes it easier to create React components.

Example:
```jsx
const element = <h1>Hello, React!</h1>;
```

#### 4. What is the difference between functional and class components?
**Answer:**

|Feature|Functional Component|Class Component|
|---|---|---|
|Syntax|Function-based|ES6 Class-based|
|State|Uses `useState` hook|Uses `this.state`|
|Lifecycle|Uses `useEffect`|Uses lifecycle methods (`componentDidMount`, etc.)|
|Performance|More optimized|Less optimized|

Example of a **functional component**:
```jsx
function Hello() {
  return <h1>Hello, World!</h1>;
}
```

Example of a **class component**:
```jsx
class Hello extends React.Component {
  render() {
    return <h1>Hello, World!</h1>;
  }
}
```

---
### **React Hooks and State Management**
#### 5. What are React Hooks?
**Answer:**  
React Hooks are functions that allow functional components to use state and lifecycle features. Some commonly used hooks are:

- `useState` → Manages state in functional components.
- `useEffect` → Handles side effects (like API calls).
- `useContext` → Provides global state management.
- `useRef` → Accesses DOM elements or persists values across renders.

#### 6. How does `useState` work?
**Answer:**  
`useState` is a React Hook that lets you add state to a functional component.

Example:
```jsx
import React, { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0); // Initial state = 0

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}
```

---
### **Component Lifecycle and Side Effects**
#### 7. What is the difference between `useEffect` and lifecycle methods in class components?

**Answer:**
- In class components, lifecycle methods are used (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`).
- In functional components, `useEffect` is used for **side effects** (API calls, event listeners, etc.).

Example using `useEffect`:
```jsx
import React, { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Component mounted or updated!");

    return () => {
      console.log("Component unmounted!"); // Cleanup function
    };
  }, [count]); // Runs when `count` changes

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}
```

---
### **Handling Events and Forms**
#### 8. How do you handle events in React?
**Answer:**  
Events in React are handled similarly to HTML but use camelCase for event names.
Example:
```jsx
function ButtonClick() {
  function handleClick() {
    alert("Button clicked!");
  }

  return <button onClick={handleClick}>Click Me</button>;
}
```

---
### **Routing in React**
#### 9. What is React Router?
**Answer:**  
React Router is a library for handling navigation in React applications.
Example of using `react-router-dom`:
```jsx
import { BrowserRouter as Router, Route, Switch } from "react-router-dom";
import Home from "./Home";
import About from "./About";

function App() {
  return (
    <Router>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
      </Switch>
    </Router>
  );
}
```

---

### **Performance Optimization**
#### 10. What is the purpose of `React.memo`?
**Answer:**  
`React.memo` is used to optimize performance by preventing unnecessary re-renders of functional components.
Example:
```jsx
const MyComponent = React.memo(function MyComponent({ name }) {
  console.log("Rendering MyComponent");
  return <p>Hello, {name}</p>;
});
```
This will only re-render if the `name` prop changes.

---
### **Advanced Questions for Interns**

#### 11. What is the difference between controlled and uncontrolled components?
**Answer:**
- **Controlled Component** → State is controlled by React.
- **Uncontrolled Component** → Uses the DOM’s state.
Example of a **controlled component** (recommended approach):
```jsx
function ControlledInput() {
  const [value, setValue] = useState("");

  return (
    <input value={value} onChange={(e) => setValue(e.target.value)} />
  );
}
```

Example of an **uncontrolled component**:
```jsx
function UncontrolledInput() {
  const inputRef = useRef();

  function handleSubmit() {
    alert(inputRef.current.value);
  }

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```
