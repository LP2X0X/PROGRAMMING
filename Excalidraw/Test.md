```mermaid
---
title: Surface
---
classDiagram
    class SurfaceInspectionItem
    SurfaceInspectionItem : +String owner
    SurfaceInspectionItem : +Bigdecimal balance
    SurfaceInspectionItem : +deposit(amount)
    SurfaceInspectionItem : +withdrawal(amount)
```