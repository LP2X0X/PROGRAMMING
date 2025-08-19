---
tags:
  - react
  - note
  - fundamental
---

### Component size **matters**
![[Pasted image 20250808191136.png|center]]

----

### How to split a UI into components
![[Pasted image 20250808191704.png|center]]

----

### When to create a new component

- You need a new component when:

| **#** | **Category**                         | **Questions to Ask**                                                                                                                                                                                            |
| ----- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | Logical separation of content/layout | - Does the component contain pieces of content or layout that **don’t belong together**?                                                                                                                        |
| 2     | Reusability                          | - Is it possible to reuse part of the component?- Do you **want** or **need** to reuse it?                                                                                                                      |
| 3     | Responsibilities / complexity        | - Is the component doing too **many different things**?- Does it rely on too **many props**?- Does it have too **many pieces of state** and/or effects?- Is the code, including JSX, too **complex/confusing**? |
| 4     | Personal coding style                | - Do you prefer **smaller** functions/components?                                                                                                                                                               |

```ad-tip
When in doubt, start with a relatively big component, then split it into smaller components as it becomes necessary. Skip complexity/reusability checks early unless you know you need them.
```

----

### General Guidelines

![[Pasted image 20250808192557.png|center]]
![[Pasted image 20250808192722.png|center]]