---
tags: react, internal, advance
---

![[Pasted image 20250811191454.png|center|700]]

## 1. Render is triggered

- There can be two situations that trigger renders:
	- **Initial render** of the application.
	- **State is updated** in one or more component instances (*re-render*).

```ad-note
The render process is triggered for the entire application.
```

## 2. Render phase
- At the beginning of the render phase, React will go through the entire component tree, take all the component instances that triggered a re-render and actually [[Render|render]] them, which simply means to call the corresponding component functions that we have written in our code. 
- This will create updated React elements which altogether make up the so-called [[Virtual Dom|virtual DOM]].
- After a state update, the new virtual DOM that React creates is compared against the current Fiber tree — the data structure representing the UI before the update. This comparison process, called [[Reconciliation|reconciliation]], is handled by React’s reconciler known as **Fiber**. 
- The reconciliation produces an updated [[Fiber|Fiber tree]], which React will later use to determine the exact changes needed and apply them to the real DOM.

## 3. Commit phase
- React writes to the DOM: insertions, deletions and updates (ist of updates are *flushed* to the DOM).
- Committing is synchronous: DOM updated in one go, it can't be interrupted. This is necessary so that the DOM never shows partial results, ensuring a consistent UI (in sync with state at all time).
> That's the whole point of dividing the entire process of rendering and committing in the first place so that the rendering can be paused, resumed, and discarded. The result of that rendering can then be flushed in one go.
- After this phase complete, the **workInProgress** fiber tree become the next fiber tree for the next render cycle.

```ad-note
This phase is handle by a [[Renderer]].
```

## 4. Browser Paint phase
- Updated UI on screen.