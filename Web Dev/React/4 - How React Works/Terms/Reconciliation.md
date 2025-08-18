---
tags: react, term, fundamental
---

- **React reconciliation** is the process React uses to figure out _how to update the actual DOM_ when your application’s state or props change. In another term, it means deciding which DOM elements actually need to be inserted, deleted or updated in order to reflect the latest state change.

![[Pasted image 20250812200611.png|center|700]]

---

### **Why reconciliation matters**

- Without reconciliation, every change would require rebuilding the entire DOM — which is slow.
- With reconciliation, React minimizes DOM operations and keeps updates efficient.

---

**Quick analogy:**

```ad-summary
Think of React reconciliation like an editor comparing two versions of a document — it doesn’t rewrite the whole thing, it just changes the lines that are different.
```