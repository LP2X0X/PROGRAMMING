---
tags: 
 - typescript
 - narrowing
 - type
 - union
 - overview
---

Narrowing is the process of moving from a broad type (like `string | number`) to a specific type (like `string`). TypeScript's Control Flow Analysis does this automatically when you write standard JavaScript checks.

Here are all the ways to narrow a union in TypeScript.

---

### 1. `typeof` (Primitives)

The most common way to check for primitives (`string`, `number`, `boolean`, `symbol`).

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input; // Narrowed to number
  }
  return padding + input; // Narrowed to string
}
```

### 2. `instanceof` (Classes & Prototypes)

Used for narrowing values that are instances of a class (like `Date`, `Array`, or custom classes).

```ts
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString()); // Narrowed to Date
  } else {
    console.log(x.toUpperCase()); // Narrowed to string
  }
}
```

### 3. The `in` Operator (Property Existence)

Checks if an object possesses a specific property key. Useful when objects don't share a common "tag".

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim(); // Narrowed to Fish
  } else {
    animal.fly();  // Narrowed to Bird
  }
}
```

### 4. Discriminated Unions (The "Gold Standard")

When every type in a union has a shared literal property (usually called `type` or `kind`), TypeScript can narrow based on that value.

```ts
type Success = { status: "success"; data: string };
type Failure = { status: "failed"; error: Error };

function handle(response: Success | Failure) {
  if (response.status === "success") {
    console.log(response.data); // Narrowed to Success
  } else {
    console.log(response.error); // Narrowed to Failure
  }
}
```

### 5. Equality Narrowing (`===`, `!==`)

Checking a variable against a specific value automatically narrows the type if that value is a literal type.

```ts
function setAlign(align: "left" | "right" | "center" | number) {
  if (align === "left") {
    // TS knows 'align' is specifically the string literal "left"
  } else if (align === 0) {
    // TS knows 'align' is the number 0
  }
}
```

### 6. Truthiness Check (`&&`, `!`, `if`)

Used primarily to filter out `null` or `undefined`.

```ts
function print(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    // 'strs' is string[]. 
    // The check filtered out null (falsy) AND string (not an object)
    for (const s of strs) { console.log(s); }
  }
}
```

### 7. User-Defined Type Guards (`is`)

When the logic is too complex for inline checks, you write a function that returns a **Type Predicate** (`arg is Type`).

```ts
type Fish = { swim: () => void };

// The return type 'pet is Fish' is the magic part
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function getPet(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // Narrowed to Fish
  } else {
    pet.fly(); // Narrowed to Bird
  }
}
```

### 8. Assertion Functions (`asserts`)

Similar to type guards, but instead of returning `true/false`, they throw an error if the check fails. This narrows the type for the remainder of the scope.

```ts
function assertIsString(val: any): asserts val is string {
  if (typeof val !== "string") {
    throw new Error("Not a string!");
  }
}

function process(val: string | number) {
  assertIsString(val);
  // From this line down, 'val' is guaranteed to be a string
  console.log(val.toUpperCase()); 
}
```

### Summary Table

| **Method**                | **Use Case**                                               |
| ------------------------- | ---------------------------------------------------------- |
| **`typeof`**              | Checking primitives (`string`, `number`).                  |
| **`instanceof`**          | Checking Class instances (`Date`, `Error`).                |
| **`in`**                  | Checking objects by property keys.                         | 
| **Equality (`===`)**      | Checking specific literal values.                          |
| **Discriminated Union**   | Checking objects via a shared `type` tag (Best for state). |
| **Type Predicate (`is`)** | Custom/Complex logic extraction.                           |
| **Assertion (`asserts`)** | Validating data (often from APIs) that throws errors.      |