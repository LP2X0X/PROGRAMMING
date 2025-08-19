---
tags:
  - js
  - term
  - fundamental
---

```js
child → parent → Object.prototype → null
```

- When you try to access a property of an object, JavaScript looks up the **prototype chain** until it finds the property (or reaches null and returns undefined).