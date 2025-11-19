---
tags: 
 - typescript
 - type
 - alias
---

### 🔹 Definition

A **type alias** gives a **custom name** to a type —  
it can represent a **primitive**, **union**, **object**, **function**, or **complex** type.

```ts
type MyString = string;
type ID = number | string;
type User = { name: string; age: number };
```

You can think of it like a _nickname_ for a type — reusable and easy to read.

---

### 🔹 Syntax

```ts
type AliasName = TypeDefinition;
```

✅ Examples:

```ts
type Point = { x: number; y: number };
type Status = "loading" | "success" | "error";
type Response = string | null;
type Callback = (msg: string) => void;
```

---

### 🔹 Why Use Type Aliases

|Benefit|Description|
|---|---|
|✅ Readability|Complex types become easy to understand|
|✅ Reusability|Reuse across multiple variables or functions|
|✅ Simplifies unions/intersections|Name large combined types|
|✅ Consistent updates|Change the alias once → updates everywhere|

---

### 🔹 Aliases for Union Types

```ts
type ID = string | number;

let userId: ID = "abc123";
userId = 101; // ✅ both allowed
```

---

### 🔹 Aliases for Object Types

```ts
type User = {
  id: number;
  name: string;
  email?: string;
};

const u: User = { id: 1, name: "Alice" };
```

---

### 🔹 Aliases for Function Types

```ts
type Greet = (name: string) => string;

const sayHello: Greet = (name) => `Hello, ${name}`;
```

---

### 🔹 Combining with Union or Intersection

```ts
type Cat = { meow: () => void };
type Dog = { bark: () => void };

type Pet = Cat | Dog; // Union
type PetOwner = { name: string } & Pet; // Intersection
```

---

### 🔹 Recursive Type Aliases

Type aliases can reference themselves for recursive structures:

```ts
type TreeNode = {
  value: string;
  children?: TreeNode[];
};
```

---

### 🔹 Type Aliases vs Interfaces

|Feature|Type Alias|Interface|
|---|---|---|
|Can describe primitives, unions, functions|✅ Yes|❌ No|
|Can be extended/merged|⚠️ No (but can combine with `&`)|✅ Yes|
|Can describe objects|✅ Yes|✅ Yes|
|Duplicate declarations merge|❌ No|✅ Yes|
|Preferred for|Complex, union, or function types|Object shapes, API contracts|

---

### 🔹 Extending with Intersections

```ts
type Animal = { name: string };
type Bear = Animal & { honey: boolean };

const b: Bear = { name: "Pooh", honey: true };
```

---

### 🧠 Pitfalls

|Pitfall|Description|Fix|
|---|---|---|
|❌ Overusing for simple cases|Adds unnecessary abstraction|Use inline types if short|
|❌ Confusing alias with value|Type aliases only exist at compile time|Can't use at runtime|
|❌ Expecting interface merging|Aliases can’t merge declarations|Use interface if merging is needed|

---

### 💡 Tips

- Prefer `type` for **unions**, **intersections**, and **functions**.
    
- Prefer `interface` for **extending objects** and **declaration merging**.
    
- Use PascalCase for alias names (`UserInfo`, not `userInfoType`).
    

---

### 🧭 Example Summary

```ts
type ID = string | number;
type Role = "admin" | "editor" | "viewer";

type User = {
  id: ID;
  name: string;
  role: Role;
  login: (password: string) => boolean;
};

const user: User = {
  id: 1,
  name: "Alice",
  role: "admin",
  login: (pw) => pw === "1234",
};
```

---

### ✅ Quick Recap

|Concept|Description|Example|
|---|---|---|
|**Definition**|Alias a type with a new name|`type ID = string|
|**Use for**|Complex, reusable, or combined types|`type Pet = Dog|
|**Extend with**|`&` (intersection)|`type Full = Base & Extra`|
|**Not for**|Runtime values|`type` disappears after compile|