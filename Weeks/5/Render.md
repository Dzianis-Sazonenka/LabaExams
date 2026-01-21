
---

## Level 1: Foundations of Rendering
**Topics:** Virtual DOM vs. Real DOM, React Elements, The Concept of Rendering.

### 1. Expanded Concept Explanations

**1. Virtual DOM vs. Real DOM:**
The **Real DOM** (Document Object Model) is the browser's direct interface for drawing the screen. Manipulating it is slow because every change triggers layout recalculations. The **Virtual DOM** is a lightweight JavaScript representation of the DOM. React updates this object first, compares it to the previous version, and only touches the Real DOM where strictly necessary.

**2. React Elements:**
A React Element is a plain JavaScript object that describes a DOM node (e.g., `{ type: 'div', props: { className: 'box' } }`). It is not an actual DOM node. "Rendering" in React is the process of transforming these lightweight descriptions into actual UI pixels.

### 2. Variant Question
"Many developers say 'React is fast because of the Virtual DOM.' Is the Virtual DOM inherently faster than the Real DOM? If not, what actually makes React performance efficient?"

### 3. Correct Answer
No, the Virtual DOM is not "faster" than the Real DOM (JavaScript operations are fast, but adding a layer is technically overhead). React is efficient because it **batches** updates and uses a **diffing algorithm** to minimize the number of costly operations (reflows/repaints) performed on the Real DOM. It avoids "thrashing" the browser layout.

### 4. Practical Code Snippet
```javascript
// A React Element (Just an object)
const element = React.createElement('h1', { className: 'greeting' }, 'Hello');

// Roughly equals:
// {
//   type: 'h1',
//   props: { className: 'greeting', children: 'Hello' }
// }
```

### 5. Coding Task
**Question:** Without using JSX, write the code to create a nested React structure: a `div` containing a `span` with the text "Hello World".
**Answer:**
```javascript
React.createElement(
  'div',
  null,
  React.createElement('span', null, 'Hello World')
);
```

---

## Level 2: ReactDOM Usage
**Topics:** Why we use ReactDOM, How to use ReactDOM, `createRoot` vs. `render`.

### 1. Expanded Concept Explanations

**1. Why we use ReactDOM:**
React (the core library) deals with components, props, state, and effects. It doesn't know about browsers. **ReactDOM** is the specific library that serves as the "glue" between React's logic and the web browser's DOM. We use it to tell React *where* to put the application on the HTML page.

**2. How we use ReactDOM:**
Since React 18, we use the `client` API. The entry point is usually `createRoot`. We select a "root" DOM node (usually an empty `div` in `index.html`) and tell React to take control of it.

### 2. Variant Question
"In React 18, we switched from `ReactDOM.render` to `ReactDOM.createRoot`. What major feature did this switch enable, and how does it change how React updates the screen?"

### 3. Correct Answer
This switch enabled **Concurrent Features** (Concurrent Mode). `createRoot` creates a root that supports concurrent rendering, allowing React to prepare multiple versions of the UI in the background, pause/resume work, and avoid blocking the main thread during heavy updates.

### 4. Practical Code Snippet
```javascript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

// React 18+ syntax
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
```

### 5. Coding Task
**Question:** You are migrating a legacy app from React 16 to React 18. Rewrite the mounting code below to use the new API.
*Legacy:* `ReactDOM.render(<App />, document.getElementById('root'));`
**Answer:**
```javascript
// New React 18 API
const container = document.getElementById('root');
const root = ReactDOM.createRoot(container);
root.render(<App />);
```

---

## Level 3: Architecture (The Split)
**Topics:** The React package vs. The ReactDOM package, Multi-platform targets (React Native).

### 1. Expanded Concept Explanations

**1. Why ReactDOM was moved to a separate library:**
Historically, React included DOM logic. In version 0.14, the team split them.
* **`react` package (The Reconciler):** Contains the core logic: Components, Hooks, State, and the diffing algorithm (Reconciliation). It computes *what* needs to change.
* **`react-dom` package (The Renderer):** Contains the logic for applying changes to the web browser.

**2. The Benefit (Universal React):**
This separation allows the core `react` package to be platform-agnostic. The same React component logic can drive:
* **Web:** via `react-dom`
* **Mobile (iOS/Android):** via `react-native`
* **VR:** via `react-360`
* **PDFs:** via `react-pdf`
  The core logic calculates the tree; the specific renderer paints it.

### 2. Variant Question
"If I am building a React Native mobile app, do I install `react-dom`? Explain your answer based on the separation of concerns between Reconciler and Renderer."

### 3. Correct Answer
No, you do not install `react-dom`. You install `react` (logic) and `react-native` (renderer). `react-dom` contains code specific to the browser (like `document.createElement` or `addEventListener`), which does not exist in a mobile environment. `react-native` provides the bridge to mobile UI primitives.

### 4. Practical Code Snippet
```javascript
// The import paths show the separation:

// 1. Core Logic (Shared everywhere)
import { useState, useEffect } from 'react'; 

// 2. Renderer (Platform specific)
// Web:
import ReactDOM from 'react-dom/client';
// OR Mobile:
// import { AppRegistry } from 'react-native';
```

### 5. Coding Task
**Question:** In a theoretical "React for Terminal" renderer, what would be the equivalent of `ReactDOM.createRoot`? Write a pseudo-code function call illustrating the pattern of passing a "Container" to a "Renderer".
**Answer:**
```javascript
import ReactTerminal from 'react-terminal-renderer';
import App from './App';

// We target the terminal screen instead of a DOM node
const terminalRoot = ReactTerminal.createRoot(process.stdout);
terminalRoot.render(<App />);
```

---

## Level 4: Internal Mechanics
**Topics:** Fiber Architecture, Reconciliation Phases, Diffing Algorithm.

### 1. Expanded Concept Explanations

**1. How it works inside (The Fiber Reconciler):**
React's internal engine is called **Fiber**. It splits rendering work into chunks (units of work).
* **The Render Phase:** React traverses the component tree, calls render functions, and compares the new Virtual DOM with the old one (Diffing). This phase is pure calculation and can be paused/aborted.
* **The Commit Phase:** React takes the calculated changes (the "effect list") and applies them to the Real DOM. This phase is synchronous and cannot be interrupted.

**2. Key Mechanics:**
* **Reconciliation:** The process of syncing the Virtual DOM with the Real DOM.
* **Keys:** Used to identify items in lists so React knows if an item was moved, added, or deleted, rather than re-rendering the whole list.

### 2. Variant Question
"What is the 'Double Buffering' technique in React's Fiber architecture, and how does it relate to the `current` and `workInProgress` trees?"

### 3. Correct Answer
React maintains two trees in memory:
1.  **Current Tree:** Represents what is currently visible on the screen.
2.  **workInProgress Tree:** A draft tree React builds in the background during updates.
    React performs all calculations on the `workInProgress` tree. Once the tree is complete and valid, React swaps the pointers (making `workInProgress` the new `current`), instantly updating the UI. This prevents the user from seeing a half-rendered state.

### 4. Practical Code Snippet
```javascript
// Conceptual view of a Fiber Node
const fiber = {
  type: 'div',
  stateNode: document.createElement('div'), // Link to Real DOM
  child: null,      // First child
  sibling: null,    // Next sibling
  return: null,     // Parent
  flags: Placement, // Side effect (e.g., needs to be added to DOM)
  alternate: null,  // Link to the corresponding fiber in the 'other' tree
};
```

### 5. Coding Task
**Question:** React uses a heuristic O(n) algorithm for diffing instead of the standard O(n³) tree comparison. What are the two main assumptions React makes to achieve this speed?
**Answer:**
1.  Two elements of different types (e.g., changing a `<div>` to a `<span>`) will produce different trees. React will simply tear down the old tree and build the new one from scratch.
2.  The developer can hint at which child elements may be stable across different renders using a `key` prop.