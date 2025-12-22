---
tags: 
 - typescript
 - object
 - type
 - annotation
---

An **object type annotation** defines the **shape (structure)** of an object — what properties it has, and their types.

```ts
let user: { name: string; age: number } = {
  name: "Alice",
  age: 25,
};
```

---

### 🔹 Syntax

```ts
let variableName: {
  propertyName: PropertyType;
  anotherProperty?: PropertyType; // optional
};
```

Example:

```ts
let car: { brand: string; year: number; electric?: boolean };
```

In JavaScript, if you access a property that doesn’t exist, you’ll get the value `undefined` rather than a runtime error. Because of this, when you _read_ from an optional property, you’ll have to check for `undefined` before using it.

```ts
function printName(obj: { first: string; last?: string }) {  
// Error - might crash if 'obj.last' wasn't provided! 
 console.log(obj.last.toUpperCase());'obj.last' is possibly 'undefined'.
  if (obj.last !== undefined) {    
	  // OK    
	  console.log(obj.last.toUpperCase());  
  }   
// A safe alternative using modern JavaScript syntax:  
console.log(obj.last?.toUpperCase());}
```

---

### 🔹 Optional and readonly properties

Object types can also specify that some or all of their properties are _optional_. To do this, add a `?` after the property name:

```ts
type Book = {
  readonly id: number;
  title: string;
  author?: string;
};

let novel: Book = { id: 1, title: "1984" };
novel.id = 2; // ❌ cannot assign to readonly property
```

---

### 🔹 Index signatures

Allow arbitrary property names:

```ts
let dictionary: { [key: string]: string };
dictionary = { hello: "xin chào", bye: "tạm biệt" };
```

---

### 🔹 Nested objects

```ts
type User = {
  name: string;
  address: {
    city: string;
    zip: number;
  };
};
```

---

### 🔹 Function properties

```ts
type Greeter = {
  greet: (name: string) => void;
};

const obj: Greeter = {
  greet(name) {
    console.log("Hello " + name);
  },
};
```

---

### 💡 Tips

|Concept|Description|
|---|---|
|Use type alias or interface|Cleaner for reuse|
|Use `?` for optional fields|Makes property not required|
|Use `readonly`|Prevent modification|
|Prefer interfaces for extension|Easier inheritance syntax|

---

### 🧠 Example

```ts
interface User {
  id: number;
  name: string;
  isAdmin?: boolean;
}

function printUser(u: User) {
  console.log(`${u.name} (${u.id})`);
}
```