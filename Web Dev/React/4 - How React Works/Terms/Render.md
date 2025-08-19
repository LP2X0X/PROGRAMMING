---
tags:
  - react
  - term
  - fundamental
---

- In React, **rendering** at its core means _calling your component functions_ (for function components) or _calling the_ _render()_ _method_ (for class components) to produce React elements.

- But there are two important clarifications:
	1. **React rendering ≠ DOM updating**
	    - When React “renders” a component, it’s _just_ executing the function to get a new description of the UI (React elements).
	    - Those React elements are _plain JavaScript objects_ describing what the UI should look like — not actual DOM nodes yet.
	    - After rendering, React compares the new elements to the previous ones (the “reconciliation” step) and updates the DOM only where necessary.
	2. **Rendering may happen without visual changes**
	    - React might call a component function multiple times even if the output is the same as before (e.g., due to a parent re-rendering).
	    - If nothing changes after reconciliation, the DOM isn’t touched, but the render function was still called.

</br>

- So if we were to phrase it precisely:
```ad-summary
Rendering in React is the process of executing components to produce a tree of React elements, which React then uses to determine how to update the actual UI.
```
