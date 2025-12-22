---
tags: 
 - typescript
 - compiler
 - flag
---

- By default, values like `null` and `undefined` are assignable to any other type. This can make writing some code easier, but forgetting to handle `null` and `undefined` is the cause of countless bugs in the world - some consider it a [billion dollar mistake](https://www.youtube.com/watch?v=ybrQvs4x0Ps)! 
- The [`strictNullChecks`](https://www.typescriptlang.org/tsconfig#strictNullChecks) flag makes handling `null` and `undefined` more explicit, and _spares_ us from worrying about whether we _forgot_ to handle `null` and `undefined`.