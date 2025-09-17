---
tags: 
 - css
 - pitfall
 - height
 - width
---

When you write height: 100%, the browser looks at the **parent’s content box height**, not the parent’s total rendered size.

So:

- **Content box** = the area defined by height (or resolved from flex/grid sizing).
    
- **Padding** (and border, margin) sit _outside_ that content box.
    
- A child with height: 100% can only ever fill the parent’s content box, not include the padding or border.
    

👉 Padding never gets included in % height calculations.