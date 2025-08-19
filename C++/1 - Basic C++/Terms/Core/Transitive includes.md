---
tags:
  - cpp
  - term
  - fundamental
---

- When your source (.cpp) file \#includes a header file, you’ll also get any other header files that are \#included by that header (and any header files those include, and so on). These additional header files are sometimes called **transitive includes**, as they’re included implicitly rather than explicitly.

```ad-tip
Each file should explicitly #include all of the header files it needs to compile. Do not rely on headers included transitively from other headers.
```