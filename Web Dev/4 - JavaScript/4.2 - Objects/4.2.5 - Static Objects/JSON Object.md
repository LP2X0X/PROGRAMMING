---
tags: 
 - js
 - object
 - static
 - special
---

## 1. The Core Methods

There are only two methods, but they are more powerful than most developers realize because of their hidden optional arguments.

### A. `JSON.stringify(value, replacer?, space?)`

Converts a JavaScript value into a JSON string.

**1. The Basic Usage**

```js
const user = { name: "Alice", id: 1 };
const str = JSON.stringify(user); 
// Output: '{"name":"Alice","id":1}'
```

2. The "Replacer" (2nd Argument)

This acts as a filter or transformer for the output. It can be an Array or a Function.

- **As an Array (The Allowlist):** Only keeps the properties listed.
    
    ```js
    const user = { name: "Alice", id: 1, age: 25, admin: false };
    // Only keep name and age
    console.log(JSON.stringify(user, ['name', 'age'])); 
    // Output: '{"name":"Alice","age":25}'
    ```
    
- **As a Function (The Transformer):** Runs for every key-value pair.
    
    ```js
    const user = { name: "Alice", age: 25 };
    
    const str = JSON.stringify(user, (key, value) => {
      // Filter out 'age'
      if (key === 'age') return undefined;
      // Convert name to uppercase
      if (key === 'name') return value.toUpperCase();
      return value; // Return everything else unchanged
    });
    // Output: '{"name":"ALICE"}'
    ```
    

3. The "Space" (3rd Argument)

Used for "Pretty Printing".

- **Number:** Indents with that many spaces (max 10).
    
- **String:** Indents with that character (e.g., `\t`).
    

```js
console.log(JSON.stringify(user, null, 2));
/* Output:
{
  "name": "Alice",
  "id": 1
}
*/
```

---

### B. `JSON.parse(string, reviver?)`

Parses a JSON string back into a JavaScript object.

**1. The Basic Usage**

```js
const data = '{"x": 10}';
const obj = JSON.parse(data); // { x: 10 }
```

2. The "Reviver" (2nd Argument)

This allows you to transform data as it is being parsed. This is extremely useful for Dates, which JSON does not natively support (JSON stores dates as strings).

```js
const data = '{"event": "Party", "date": "2023-10-25T12:00:00.000Z"}';

const obj = JSON.parse(data, (key, value) => {
  // Check if the key is 'date', if so, turn it into a real Date object
  if (key === 'date') return new Date(value);
  return value;
});

console.log(obj.date.getFullYear()); // 2023 (It's a real Date object now!)
```

---

## 2. The `toJSON` Method

You can customize how your _own_ objects get serialized by adding a method named `toJSON` to them. If `JSON.stringify` sees this method, it calls it instead of doing the default serialization.

**Use Case:** Hiding sensitive data (passwords) or simplifying complex objects.

```js
const user = {
  name: "Alice",
  password: "super_secret_password",
  dateCreated: new Date(),
  
  // Custom serializer
  toJSON() {
    return {
      name: this.name,
      // We explicitly leave out the password
      meta: "User created via API"
    };
  }
};

console.log(JSON.stringify(user));
// Output: '{"name":"Alice","meta":"User created via API"}'
```

---

## 3. The "Silent Data Loss" (Crucial Limitations)

JSON is **language-agnostic**, meaning it only supports data types that exist in almost every programming language (String, Number, Boolean, Array, Object, Null).

JavaScript has types that JSON does **not** support. When you `stringify` them, specific things happen:

| **JS Type**            | **Behavior in JSON.stringify**                    |
| ---------------------- | ------------------------------------------------- |
| **`Function`**         | Skipped (removed from object).                    |
| **`undefined`**        | Skipped (removed from object).                    |
| **`Symbol`**           | Skipped (removed from object).                    |
| **`Date`**             | Converted to ISO String (`"2023-..."`).           |
| **`Map` / `Set`**      | Converted to empty object `{}`. **(Major Trap!)** |
| **`NaN` / `Infinity`** | Converted to `null`.                              |
| **Circular Ref**       | Throws `TypeError` (crashes).                     |

**Example of Loss:**

```js
const data = {
  a: "hello",
  b: undefined,        // Will vanish
  c: () => "hi",       // Will vanish
  d: new Map([['x',1]]) // Becomes {}
};

console.log(JSON.stringify(data));
// Output: '{"a":"hello","d":{}}'
```

---

## 4. The "Deep Clone" Pattern

For years, developers used the `JSON` object to create a "Deep Copy" of an object (breaking reference links).

**The Old Hack:**

```js
const original = { a: 1, b: { c: 2 } };
const clone = JSON.parse(JSON.stringify(original));
```

- _Pros:_ Simple, one line.
    
- _Cons:_ Fails on `Date` (becomes string), `undefined`, `Map`, `Set`, and `RegExp`.
    

The Modern Replacement:

Use structuredClone(original). It is built into the browser, faster, and handles Dates, Maps, and Sets correctly.

---

## 5. Strictness & Errors

`JSON.parse()` is notoriously fragile.

1. **Quotes:** Keys _must_ be double-quoted. `{ x: 1 }` is valid JS, but invalid JSON. It must be `{ "x": 1 }`.
    
2. **Trailing Commas:** JSON does _not_ allow trailing commas. `[1, 2, ]` will throw an error.
    
3. **Comments:** JSON does not support comments (`//` or `/* */`).
    

Pro Tip for debugging:

Always wrap JSON.parse in a try...catch block if the data comes from an external source (API or LocalStorage), or your app will crash on a single bad character.

```js
try {
  const data = JSON.parse(userInput);
} catch (e) {
  console.error("The user provided broken JSON!");
}
```