---
tags: react, distinguish, fundamental
---

![[Pasted image 20250722195150.png|center]]

|                                                                  State                                                                  |                                                                                      Prop                                                                                       |
|:---------------------------------------------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|                                                    Internal data owned by component                                                     |                                                                     External data owned by parent component                                                                     |
| Can be updated by the component itself which causes the component to be rerendered, we use this mechanism to make component interactive | Read-only. If the child component received the update prop, that component will be rerendered also which give the parent component the ability to configure the child component |
