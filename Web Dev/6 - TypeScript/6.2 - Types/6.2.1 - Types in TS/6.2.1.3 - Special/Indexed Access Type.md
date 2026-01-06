---
tags: 
 - typescript
 - type
 - special
---

**Indexed Access Types** in TypeScript allow you to look up a specific property on another type:

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"];
     
// type Age = number;
```

Think of it like accessing a value in a JavaScript object using `obj["key"]`, but instead of getting a **value**, you are getting a **type**.

---

### 1. Basic Syntax

You use the `Type[Index]` syntax. The "Index" must be a type itself (usually a string literal or a union of literals).

```ts
type User = {
  id: number;
  username: string;
  address: {
    city: string;
    zip: number;
  };
};

// Accessing a single property type
type UserId = User["id"];       // type UserId = number
type UserCity = User["address"]["city"]; // type UserCity = string
```

> Note: You cannot use a variable as an index; it must be a type.
> 
> const key = "id"; type T = User\[key]; ❌ (Error)
> 
> type Key = "id"; type T = User\[Key]; ✅

````ad-note
**You may index with `number` when the type has a numeric index signature or is a tuple.**

#### 1. Tuples (Most Common Case)

Tuples have **known numeric positions**, so numeric indexing is allowed.

```ts
type Params = [number, string, boolean];

type First = Params[0]; // number
type Second = Params[1]; // string
type Any = Params[number]; // number | string | boolean
```

`Params[number]` means:

> “Give me the union of all element types in this tuple.”

#### 2. Arrays

Arrays define a numeric index signature:

```ts
type Numbers = number[];

type Item = Numbers[number]; // number
```

Equivalent to:

```ts
type Item = Array<number>[number];
```

#### 3. Function Parameters (via `Parameters<T>`)

Because `Parameters<T>` returns a **tuple**, numeric indexing works:

```ts
type Fn = (x: string, y: number) => void;

type Args = Parameters<Fn>;

type FirstArg = Args[0]; // string
type ArgUnion = Args[number]; // string | number
```

#### 4. Objects with Numeric Index Signatures

```ts
type Dict = {
  [key: number]: string;
};

type Value = Dict[number]; // string
```

This is less common but valid.

#### When You **Cannot** Use Indexes

❌ Plain objects without numeric index signatures

```ts
type User = {
  id: number;
  name: string;
};

type X = User[0]; // Error
```

Because:

- `0` is not a key of `User`
    
- Object keys are `"id" | "name"`
    

#### Why `T[number]` Is So Useful

It is a **general pattern** for extracting value unions:

```ts
type ValueOf<T> = T[keyof T];     // objects
type ElementOf<T> = T[number];   // arrays / tuples
```

#### Mental Model

|Type kind|Valid index|
|---|---|
|Object|`"key"` or `keyof T`|
|Tuple|numeric literal (`0`, `1`, …)|
|Array|`number`|
|Dict|index signature key|

#### One-Line Summary

> **Use numeric indexed access (`T[number]`) only for tuple-like or array-like types; use property names or `keyof` for object types.**
````

---

### 2. Accessing Multiple Types (Unions)

You can pass a union of strings to the index to get a union of those property types.

```ts
type UserInfo = User["id" | "username"]; 
// type UserInfo = number | string
```

If you use `keyof`, you can get a union of **all** property types within that object:

```ts
type AllUserValues = User[keyof User];
// type AllUserValues = number | string | { city: string; zip: number; }
```

---

### 3. Indexed Access with Arrays

Indexed access is extremely powerful for extracting the type of elements inside an array. You use the special `number` index to say "give me the type of any element in this array."

```ts
const MyArray = [
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
];

// 1. Get the type of the whole array
type Members = typeof MyArray; 

// 2. Get the type of a single element (the object)
type Person = Members[number]; 
// type Person = { name: string; age: number; }

// 3. Get a specific property from the elements
type PersonName = Members[number]["name"]; 
// type PersonName = string
```

---

### 4. Real-World Use Case: API Responses

Imagine you have a complex API response and you want to write a function that only processes one specific part of it without redefining that type.

```ts
interface APIResponse {
  data: {
    users: { id: string; email: string }[];
    meta: { total: number };
  };
  status: number;
}

// Extract just the User object type
type UserFromAPI = APIResponse["data"]["users"][number];

function processUser(user: UserFromAPI) {
  console.log(user.email);
}
```

---

### 5. Key Behaviors & Constraints

| **Scenario**        | **Syntax**      | **Result**                                      |
| ------------------- | --------------- | ----------------------------------------------- |
| **Object Property** | `Obj["prop"]`   | The type of that property.                      |
| **Nested Property** | `Obj["a"]["b"]` | Deeply nested type access.                      |
| **Array Element**   | `Arr[number]`   | The type of the elements in the array.          |
| **Tuple Element**   | `Tuple[0]`      | The type of the element at that specific index. |

#### Important Rules:

1. **The Index is a Type:** You must use types inside the brackets. `User["id"]` is valid because `"id"` is a literal type.
    
2. **Type Safety:** If you try to access a property that doesn't exist (e.g., `User["password"]`), TypeScript will throw an error.
    

---

### Summary

Indexed Access Types allow you to maintain a **"Single Source of Truth."** Instead of manually creating `type UserId = number`, you link it directly to the master `User` type. If the `id` in the `User` interface ever changes to a `string`, your `UserId` type will update automatically.