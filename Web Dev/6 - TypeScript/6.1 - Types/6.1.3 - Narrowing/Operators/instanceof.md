---
tags: 
 - typescript
 - operator
 - type
 - guard
---

### What it does 🔍

- `instanceof` checks **whether an object was created by a specific class (constructor)**.
    
- It works at **runtime**, not just at type level.
    

```ts
obj instanceof ClassName
```

---

### Primary use case 🎯

- **Type narrowing for class-based types**
    
- Especially useful with:
    
    - Custom classes
        
    - Built-in classes (`Date`, `Error`, `Map`, etc.)
        

---

### Basic example 📦

```ts
class User {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
}

function print(value: unknown) {
  if (value instanceof User) {
    // value is now narrowed to User
    console.log(value.name);
  }
}
```

➡️ Inside the `if`, TypeScript **knows** `value` is `User`.

---

### With union types 🔀

```ts
class Dog {
  bark() {}
}

class Cat {
  meow() {}
}

function speak(pet: Dog | Cat) {
  if (pet instanceof Dog) {
    pet.bark();
  } else {
    pet.meow();
  }
}
```

🧠 `instanceof` helps TypeScript select the correct branch.

---

### Works only with classes ⚠️

✅ Works with:

- `class`
    
- Built-in constructors (`Date`, `Error`, `RegExp`)
    

❌ Does **not** work with:

- Interfaces
    
- Type aliases
    
- Plain object shapes
    

```ts
// ❌ Invalid
interface Person {
  name: string;
}

if (x instanceof Person) {} // Error
```

---

### Built-in example 🧱

```ts
function handle(err: unknown) {
  if (err instanceof Error) {
    console.log(err.message);
  }
}
```

✔ Very common in error handling.

---

### Runtime behavior ⏱

- `instanceof` checks the **prototype chain**
    
- Equivalent to:
    

```ts
ClassName.prototype.isPrototypeOf(obj)
```

---

### Comparison with other narrowing tools 🆚

|Tool|Use case|
|---|---|
|`instanceof`|Class-based objects|
|`typeof`|Primitives (`string`, `number`, `boolean`)|
|`"in"`|Property existence|
|Custom type guard|Structural checks|

---

### Key limitations 🚧

- Fails across different JS realms (e.g., iframe boundaries)
    
- Cannot check interfaces or structural types
    
- Requires constructor function at runtime
    

---

### Summary 📝

- ✅ Runtime check
    
- ✅ Narrows class instances
    
- ❌ Not for interfaces
    
- ❌ Not structural
    

> Use `instanceof` **when you control the class** and need **safe runtime narrowing**.