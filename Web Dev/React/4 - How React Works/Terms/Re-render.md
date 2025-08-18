---
tags: react, term, fundamental
---

![[Pasted image 20250811194247.png|center|500]]

- If a state update occurs in component D, React will re-render it by **calling its function again and generating a new React element for it**. This updated element will be placed into a *new* React element tree — essentially a new virtual DOM.

</br>

```ad-important
A key detail is that whenever React renders a component, it will also render all of that component’s children, regardless of whether their props have changed.
```
-  So, **if an updated component returns other components, those child components will be re-rendered as well**, cascading down the entire component tree.

- This means that if the root component (like component A) updates, the entire application will be re-rendered. While this might sound inefficient, React follows this approach because it cannot know in advance whether a child will be affected by a parent’s changes. As a safety measure, it re-renders everything — but remember, this is <u>only in the virtual DOM</u>, not the actual DOM.

![[Pasted image 20250811194523.png|center|500]]

- Since the virtual DOM is just a set of lightweight JavaScript objects, recreating it is generally not a problem for small to medium applications.