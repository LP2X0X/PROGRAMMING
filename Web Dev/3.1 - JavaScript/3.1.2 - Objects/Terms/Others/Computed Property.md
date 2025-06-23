---
tags: js, term, fundamental
---

- We can use square brackets in an object literal, when creating an object. That’s called _computed properties_.

```js
let fruit = prompt("Which fruit to buy?", "apple");

let bag = {
  [fruit]: 5, // the name of the property is taken from the variable fruit
};

alert( bag.apple ); // 5 if fruit="apple"
```

- The meaning of a computed property is simple: `[fruit]` means that the property name should be taken from `fruit`.

```js
let fruit = prompt("Which fruit to buy?", "apple");
let bag = {};

// take property name from the fruit variable
bag[fruit] = 5;
```

- We can use more complex expressions inside square brackets:

```javascript
let fruit = 'apple';
let bag = {
  [fruit + 'Computers']: 5 // bag.appleComputers = 5
};
```