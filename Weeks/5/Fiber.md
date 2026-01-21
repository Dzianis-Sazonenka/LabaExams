## Level 1: Foundations of Fiber
**Topics:** Definition of "Fiber", The Problem it Solved (Blocking Rendering).

### 1. Expanded Concept Explanations

**1. What is "Fiber"?**
React Fiber is the complete rewrite of the React core algorithm (the Reconciler), introduced in React 16. It is essentially a "virtual stack frame." While the old reconciler (Stack Reconciler) worked recursively and synchronously (blocking the main thread until the entire tree was updated), Fiber allows React to pause work, prioritize updates, and reuse previously completed work.

**2. The Problem it Solved:**
In the old "Stack" architecture, once React started rendering, it couldn't stop. If a component tree was large, the main thread would be blocked for hundreds of milliseconds, causing dropped frames and unresponsive inputs (jank). Fiber solved this by breaking rendering work into small units that can be spread out over multiple frames.

### 2. Variant Question
"Before React 16 (Fiber), if a large list update started, the browser would 'freeze' until it finished. Why did this happen in terms of the JavaScript Call Stack, and how does Fiber fundamentally change this behavior?"

### 3. Correct Answer
The old reconciler used the native JavaScript recursion stack. Once a recursive function starts, it cannot be interrupted until it returns. Fiber re-implements the stack using a custom linked list data structure (heap-based). This allows React to "yield" execution back to the browser (using `requestIdleCallback` or `Scheduler`) to handle user input, then resume where it left off.

### 4. Practical Code Snippet
```javascript
// Conceptual: The "Unit of Work" structure
const fiberNode = {
  type: 'div',
  props: { ... },
  stateNode: domElement, // Link to real DOM
  
  // The "Stack" pointers
  return: parentFiber,
  child: firstChildFiber,
  sibling: nextSiblingFiber,
};
```

### 5. Coding Task
**Question:** Explain the relationship between a "Fiber" and a "React Element". Which one is immutable and which one is mutable?
**Answer:**
* **React Element:** An immutable description of the UI (a plain object returned by `render`). It is thrown away and recreated every render.
* **Fiber:** A mutable internal object that holds component state and the DOM connection. React reuses Fiber objects (mutating their properties) to save memory allocations.

---

## Level 2: The Algorithm (In Own Words)
**Topics:** The Work Loop, Time Slicing, Yielding to the Browser.

### 1. Expanded Concept Explanations

**1. How the Algorithm Works:**
Fiber works like a collaborative multitasker. It splits the rendering process into two phases:
1.  **Render Phase (Asynchronous):** React traverses the Fiber tree, figuring out what changes. It performs work in small chunks (processing one Fiber node at a time). After every chunk, it checks: "Do I have time left in this frame?" If yes, it continues. If no, it yields control back to the browser so the browser can paint or handle clicks.
2.  **Commit Phase (Synchronous):** Once all calculations are done and the "workInProgress" tree is ready, React synchronously applies all DOM updates in one go to ensure the UI doesn't look glitchy.

**2. Time Slicing:**
This is the ability to split high-priority work (like user input) from low-priority work (like data fetching). Fiber can "interrupt" a low-priority render to process a high-priority click, then come back to the low-priority work later.

### 2. Variant Question
"Imagine React is in the middle of rendering a large table (Render Phase) and the user clicks a button. What happens to the current rendering process, and how does React ensure the button click feels responsive?"

### 3. Correct Answer
React detects the high-priority input event. It pauses (or discards) the current low-priority rendering work for the table. It immediately schedules and executes the high-priority work for the button click (updating the UI state). Once that is committed to the DOM, it restarts or resumes the lower-priority table rendering.

### 4. Practical Code Snippet
```javascript
// Pseudo-code of the Work Loop
function workLoop(deadline) {
  let shouldYield = false;
  while (nextUnitOfWork && !shouldYield) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1; // Check if frame is full
  }
  
  if (!nextUnitOfWork && workInProgressRoot) {
    commitRoot(workInProgressRoot); // Sync phase
  } else {
    requestIdleCallback(workLoop); // Continue later
  }
}
```

### 5. Coding Task
**Question:** Why is the `componentWillMount` lifecycle method unsafe in Fiber architecture?
**Answer:**
Because the **Render Phase** can be paused, restarted, or aborted, `componentWillMount` (which runs during this phase) might be called multiple times before the component actually mounts. This causes bugs if the method contains side effects (like AJAX requests).

---

## Level 3: Implementation Structure
**Topics:** The Fiber Tree (Linked List), Child/Sibling/Return pointers, Alternate Tree (Double Buffering).

### 1. Expanded Concept Explanations

**1. The Fiber Data Structure:**
Fiber implements a **Singly Linked List** traversal algorithm. Instead of standard recursion (Parent -> Children), every Fiber node points to:
* `child`: The first direct child.
* `sibling`: The next immediate sibling.
* `return`: The parent node.
  This structure allows React to pause at any node and know exactly where to go next (or go back up) without relying on the browser's call stack.



**2. Double Buffering (Current vs. WorkInProgress):**
React maintains two parallel Fiber trees:
* **current:** Mirrors what is currently on the screen.
* **workInProgress:** A draft tree where changes are computed.
  React recycles Fiber nodes from the `current` tree to build the `workInProgress` tree (saving memory). When the draft is finished, React simply swaps the root pointer: `workInProgress` becomes `current`.

### 2. Variant Question
"During a state update, does React create brand new objects for every node in the Fiber tree? If not, how does it optimize memory usage?"

### 3. Correct Answer
No. React uses **object pooling** (cloning with mutation). It looks at the `current` tree's corresponding node (`alternate`). If the node exists, React copies its properties to the `workInProgress` node and updates only the changed data. This drastic reduction in garbage collection is key to Fiber's performance.

### 4. Practical Code Snippet
```javascript
// Traversal logic pseudo-code
function performUnitOfWork(fiber) {
  // 1. Do work on current fiber (reconcile children)
  // ...
  
  // 2. Return next fiber
  if (fiber.child) {
    return fiber.child; // Go deep
  }
  let nextFiber = fiber;
  while (nextFiber) {
    if (nextFiber.sibling) {
      return nextFiber.sibling; // Go wide
    }
    nextFiber = nextFiber.return; // Go up
  }
  return null;
}
```

### 5. Coding Task
**Question:** You are debugging a memory leak. You see that a Fiber node retains a reference to an unmounted DOM node. Which property of the Fiber node holds the DOM reference, and when is it normally cleaned up?
**Answer:**
The `stateNode` property holds the reference to the DOM node (or Class Component instance). It is cleaned up when the Fiber is detached during the commit phase (via `commitDeletion`), but bugs in custom renderers or external libraries can sometimes retain these references.

---

## Level 4: Intricacies & Deep Principles
**Topics:** Effect Lists, Heuristic Diffing Assumptions, Lane Priority Model.

### 1. Expanded Concept Explanations

**1. The Effect List (Side-Effect Accumulation):**
Traversing the whole tree twice (once to diff, once to commit) is inefficient. To optimize the Commit Phase, Fiber builds a linear **Effect List** during the Render Phase. This list contains *only* the fibers that need updates (insertions, updates, deletions). The Commit Phase just iterates through this short list, ignoring unchanged nodes.

**2. The Lane Priority Model (React 18):**
React 16 used "expiration times" for priority. React 18 evolved this into **Lanes** (a bitmask system).
* Each update is assigned a "Lane" (e.g., SyncLane for clicks, TransitionLane for routing).
* Lanes allow React to work on multiple updates simultaneously without them blocking each other (e.g., suspending a transition while processing a hover effect).

### 2. Variant Question
"Explain how the `key` prop specifically helps the Fiber reconciler during the diffing of a list. What specific O(n) optimization does it unlock in the linked-list structure?"

### 3. Correct Answer
Without keys, React iterates over both lists (old and new) simultaneously. If an item is inserted at the top, React mutates every subsequent sibling. With `key`, React uses a **Hash Map** (Map data structure) to match old Fibers to new Elements by ID. This allows it to reuse the existing Fiber node (and its DOM node) just by moving it, turning an O(n^2) operation into O(n).

### 4. Practical Code Snippet
```javascript
// Concept: Lanes Bitmask
const SyncLane =       0b0000000000000000000000000000001;
const InputContinuous = 0b0000000000000000000000000000100;
const TransitionLane =  0b0000000000000000000000000010000;

// Checking priority overlap
function includesSomeLane(a, b) {
  return (a & b) !== 0;
}
```

### 5. Coding Task
**Question:** Why does the "Commit Phase" have to be synchronous? What would happen visually if React time-sliced the Commit Phase like it does the Render Phase?
**Answer:**
If the Commit Phase were asynchronous (time-sliced), the browser might paint a frame in the middle of the updates. The user would see a "tearing" UI (e.g., the top half of the list updated, but the bottom half old). Synchronous execution ensures the DOM transitions from one consistent state to the next in a single paint frame.