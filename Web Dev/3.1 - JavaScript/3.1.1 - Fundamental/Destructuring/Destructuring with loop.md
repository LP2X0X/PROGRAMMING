---
tags:
  - js
  - destructuring
---

```js
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

for (const { id, name } of users) {
  console.log(`${id}: ${name}`);
}
// 1: Alice
// 2: Bob
```