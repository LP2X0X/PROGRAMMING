---
tags: react, term, fundamental
---

- On the initial render, React processes the entire component tree and converts it into a single large React element tree. This structure, known as the **virtual DOM**, is simply a hierarchy of React elements generated from all component instances in the tree.
- Creating this virtual DOM is relatively quick and efficient because it’s essentially just a collection of plain JavaScript objects, making it inexpensive to recreate even multiple times.
![[Pasted image 20250811193853.png|center|300]]

