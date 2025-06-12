- Each destructured property can have a _default value_. The default value is used when the property is not present, or has value `undefined`. It is not used if the property has value `null`.

```js
const [a = 1] = []; // a is 1
const { b = 2 } = { b: undefined }; // b is 2
const { c = 2 } = { c: null }; // c is null
```

- The default value can be any expression. **It will only be evaluated when necessary.**

```js
const { b = console.log("hey") } = { b: 2 };
// Does not log anything, because `b` is defined and there's no need
// to evaluate the default value.
```