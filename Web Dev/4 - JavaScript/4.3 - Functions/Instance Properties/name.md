---
tags: 
 - js
 - function
 - property
 - fundamental
---

## 1. **Basic behavior**

- For **function declarations** → `name` is the identifier.
    

```js
function hello() {}
console.log(hello.name); // "hello"
```

- For **function expressions with a name** → `name` is that identifier.
    

```js
const greet = function sayHi() {};
console.log(greet.name); // "sayHi"
```

⚠️ Even if you assign it to another variable, the `.name` stays `"sayHi"`, not `"greet"`.

---

## 2. **Anonymous function expressions**

- If a function expression is **anonymous** but assigned to a variable, many engines **infer the name**.
    

```js
const anon = function() {};
console.log(anon.name); // "anon" (in modern JS; empty string in old engines)
```

- If it’s assigned as an **object property**:
    

```js
const obj = {
  method: function() {}
};
console.log(obj.method.name); // "method"
```

---

## 3. **Arrow functions**

- Always anonymous, but name inference still applies:
    

```js
const arrow = () => {};
console.log(arrow.name); // "arrow"
```

---

## 4. **Classes**

- Classes behave like functions under the hood → they have `name`.
    

```js
class Person {}
console.log(Person.name); // "Person"

const Student = class {};
console.log(Student.name); // "Student" (inferred)
```

⚠️ **Gotcha:** Anonymous class with no binding gets `""`:

```js
console.log((class {}).name); // ""
```

---

## 5. **Bound functions (`.bind`)**

- When you use `.bind`, the `name` gets `"bound "` prefixed.
    

```js
function work() {}
const boundWork = work.bind(null);
console.log(boundWork.name); // "bound work"
```

---

## 6. **Getters/Setters**

- Functions created implicitly by `get` / `set` have special names:
    

```js
const obj = {
  get value() { return 42; },
  set value(v) {}
};
console.log(Object.getOwnPropertyDescriptor(obj, "value").get.name); // "get value"
console.log(Object.getOwnPropertyDescriptor(obj, "value").set.name); // "set value"
```

---

## 7. **Default exports (ESM)**

- Default export functions/classes may end up with `""` or inferred names depending on context.
    

```js
export default function() {}
// name = "" (anonymous)

export default function myFunc() {}
// name = "myFunc"
```

---

## 8. **Read-only**

- `name` is a **read-only property** (non-writable, non-configurable).
    

```js
function foo() {}
foo.name = "bar"; 
console.log(foo.name); // "foo" (unchanged)
```

---

## ✅ Key Gotchas Recap

1. `name` can be **inferred** even for anonymous functions/classes.
    
2. `bind` always prefixes `"bound "`.
    
3. Getters/setters have `"get "` and `"set "` prefixes.
    
4. Default exports may lose the name.
    
5. It’s **read-only** → can’t be changed directly.
    