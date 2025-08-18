---
tags: react, term, advance
---

- **Render logic** is the code that lives at the top level of component function.
- It participates in describing how the component view looks like.
- It executed every time the component renders.

![[Pasted image 20250817181527.png|center|500]]

---

### Rule

```ad-important
- Components (function) must not have any [[Web Dev/React/4 - How React Works/Terms/Side Effect|side effect]] (impure). Given the same props, a component instance should always return the same JSX.
👉 Do **NOT** perform **network requests** (API calls)
👉 Do **NOT** start **timers**
👉 Do **NOT** directly **use the DOM API**
👉 Do **NOT mutate objects or variables** outside of the function scope ➡️ _This is why we can’t mutate props!_
👉 Do **NOT update state (or refs):** this will create an infinite loop!
```