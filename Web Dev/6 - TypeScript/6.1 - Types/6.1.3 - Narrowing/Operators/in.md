---
tags: 
 - typescript
 - operator
 - type
 - guard
---

JavaScript has an operator for determining if an object or its prototype chain has a property with a name: the `in` operator. TypeScript takes this into account as a way to narrow down potential types.

---

### Basic meaning in English

```ts
"prop" in obj
```

Reads as:

> “Does this object have a property called `prop`?”

---

### Why it matters in TypeScript

When you check for a property, TypeScript can **figure out which shape the object has**.

---

### Example

```ts
type User = { name: string };
type Admin = { name: string; permissions: string[] };

function printPerson(person: User | Admin) {
  if ("permissions" in person) {
    // person is Admin here
    console.log(person.permissions);
  } else {
    // person is User here
    console.log(person.name);
  }
}
```

**English interpretation:**

- “If the object has a `permissions` property, then it must be an Admin.”
    
- Otherwise, it’s a User.
    

---

### What `in` filters out

Before the check:

```ts
person = User | Admin
```

After the check:

- `if` branch → `Admin`
    
- `else` branch → `User`
    

---

### Important notes

- `in` works on **objects only**
    
- It checks for the property **anywhere on the object**, including the prototype
    
- The property name must be a **string (or symbol)**
    

---

### When to use `in`

Use `in` when:

- You are distinguishing between **object shapes**
    
- The types differ by **property presence**
    
- `typeof` is not sufficient
    

---

### One-line summary

> The `in` operator lets TypeScript narrow types by asking: _“Which object has this property?”_