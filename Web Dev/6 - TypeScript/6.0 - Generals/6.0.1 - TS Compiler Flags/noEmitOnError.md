---
tags: 
 - typescript
 - compiler
 - flag
---

- Do not emit compiler output files like JavaScript source code, source-maps or declarations if any errors were reported.

- This defaults to `false`, making it easier to work with TypeScript in a watch-like environment where you may want to see results of changes to your code in another environment before making sure all errors are resolved.

```ad-summary
`noEmitOnError` is needed to ensure that TypeScript does not produce JavaScript output when there are type errors in the codebase. Without it, the compiler may still emit `.js` files even though the program is known to be incorrect at the type level, allowing broken or unsafe code to reach runtime. Enabling `noEmitOnError` enforces a stricter build guarantee: **only type-safe code is emitted**, which is critical for CI pipelines, production builds, and maintaining confidence that compile success implies correctness.
```