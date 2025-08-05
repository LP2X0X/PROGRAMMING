---
tags: js, json
---

In JavaScript, the `toJSON()` method is **automatically called** when you use `JSON.stringify()` on an object.

> You can **customize** how an object is converted to JSON by defining your own `toJSON()` method.

---

### 🧪 Default Behavior

```js
const obj = {
  name: "Alice",
  age: 25
};

console.log(JSON.stringify(obj));
// → '{"name":"Alice","age":25}'
```

---

### 🔧 Custom `toJSON()` Method

You can define a `toJSON()` method inside your object or class to control what gets serialized:

```js
const user = {
  name: "Alice",
  password: "secret",
  toJSON() {
    // Only return what you want to include in JSON
    return {
      name: this.name // omit password
    };
  }
};

console.log(JSON.stringify(user));
// → '{"name":"Alice"}'
```

> `toJSON()` returns the value that should be stringified instead of the object itself.

---

### 🏛️ Example in a Class

```js
class User {
  constructor(name, password) {
    this.name = name;
    this.password = password;
  }

  toJSON() {
    return {
      name: this.name // omit sensitive info
    };
  }
}

const user = new User("Bob", "1234");
console.log(JSON.stringify(user));
// → '{"name":"Bob"}'
```

---

### 📌 Why Use `toJSON()`?

|Use Case|Benefit|
|---|---|
|Hide sensitive data|Remove passwords, tokens, etc.|
|Format data for APIs|Convert timestamps, add custom fields|
|Reduce payload size|Omit unnecessary properties|
|Rename fields|Match API field names|

---

### ⚠️ Key Points

- `JSON.stringify(obj)` calls `obj.toJSON()` **automatically** (if it exists).
- `toJSON()` must return a **value ready to be stringified** (usually an object).
- Doesn’t affect how you use the object in normal JavaScript — just JSON output.