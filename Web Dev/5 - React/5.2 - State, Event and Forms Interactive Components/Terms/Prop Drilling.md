---
tags:
  - react
  - term
  - fundamental
---

- **Prop drilling** is the term used in React (and other component-based frameworks) to describe the process of passing data or functions through multiple layers of components — even if some of those intermediate components don’t actually need them — just to get the values to a deeply nested child.

</br>

- **Example in plain words:** Imagine you have `App → Parent → Child → Grandchild`, and only `Grandchild` needs the `user` data. If you pass `user` from `App` to `Parent`, then from `Parent` to `Child`, then from `Child` to `Grandchild`, that’s prop drilling.

</br>

- It’s not _bad_ for small component trees, but in larger apps it makes components harder to maintain and refactor. That’s when developers use solutions like **React Context**, **state management libraries** (Redux, Zustand, Jotai), or component composition patterns to avoid it.
