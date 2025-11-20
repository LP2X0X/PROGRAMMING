---
tags: 
 - typescript
 - type
 - literal
---

## 🧠 What Is a Literal Type?

A **literal type** means a variable or value is **restricted to a specific literal value**, instead of the general type like `string` or `number`.

In other words:

> Literal types let you specify _exact values_ a variable can have.

![[Pasted image 20251107144036.png|center|500]]
 <figcaption align="center"><strong>Figure:</strong> A literal type example "Hello World"</figcaption>

It’s not much use to have a variable that can only have one value!
But by _combining_ literals into unions, you can express a much more useful concept!

```ad-note
When a constant is initialized from a primitive type, TypeScript infers it to be a literal type of the specific value assigned. However, when a constant is initialized from a non-primitive type, TypeScript only infers it to be of the same type as assigned.
```

---

### 🧩 Example

```ts
let direction: "up" | "down" | "left" | "right";
direction = "up";     // ✅ OK
direction = "left";   // ✅ OK
direction = "forward"; // ❌ Error
```

Here, `direction` can only be **one of the listed literal values**.

---

## 🧱 Types of Literal Types

### 1️⃣ **String Literal Types**

```ts
let color: "red" | "green" | "blue";
color = "red";   // ✅
color = "yellow"; // ❌
```

---

### 2️⃣ **Number Literal Types**

```ts
let diceRoll: 1 | 2 | 3 | 4 | 5 | 6;
diceRoll = 4; // ✅
diceRoll = 7; // ❌
```

---

### 3️⃣ **Boolean Literal Types**

```ts
let isDone: true;
isDone = true;  // ✅
isDone = false; // ❌
```

This is not common, but it’s useful for functions that always return a fixed boolean.

---

## 🧩 Why Use Literal Types?

They make your code:

- **Safer** (prevents invalid values)
    
- **Self-documenting**
    
- **More predictable** for function parameters and configuration objects
    

Example:

```ts
function move(direction: "up" | "down") {
  console.log(`Moving ${direction}`);
}

move("up");   // ✅
move("left"); // ❌
```

---

## 🧠 Literal Types + Union = “Enum-like” Behavior

Literal types become really powerful when **combined with union types**.

```ts
type Status = "success" | "error" | "loading";

function showStatus(status: Status) {
  console.log(status);
}

showStatus("success"); // ✅
showStatus("fail");    // ❌
```

They’re often used **instead of enums** for lightweight, flexible typing.

---

## 🧩 Literal Type Inference & `as const`

Look at this example:
```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;
 
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
// Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.

```

- In the above example req.method is inferred to be string, not "GET". Because code can be evaluated between the creation of req and the call of handleRequest which could assign a new string like "GUESS" to req.method, TypeScript considers this code to have an error.

- There are two ways to work around this:
	1. You can change the inference by adding a type assertion in either location:
		```ts
		// Change 1:
		const req = { url: "https://example.com", method: "GET" as "GET" };
		// Change 2
		handleRequest(req.url, req.method as "GET");
		```
	
	Change 1 means “I intend for req.method to always have the literal type "GET"”, preventing the possible assignment of "GUESS" to that field after. 
	Change 2 means “I know for other reasons that req.method has the value "GET"“.
	
	2. You can use as const to convert the entire object to be type literals:
		```ts
		const req = { url: "https://example.com", method: "GET" } as const;
		handleRequest(req.url, req.method);
		```
		
	The as const suffix acts like const but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like string or number.
	

By default, TypeScript **widens** literals:

```ts
let role = "admin";
// inferred as string (not "admin")
```

To make it a literal type:

```ts
const role = "admin";
// inferred as "admin"
```

💡 **Rule of thumb:**

- `let` → **widened type** (`string`, `number`, etc.)
    
- `const` → **literal type** (`"admin"`, `42`, etc.)
    

---

### 🪄 Use `as const` for literal preservation

`as const` makes **object and array literals** deeply readonly and literal-typed.

```ts
const config = {
  env: "dev",
  port: 3000
} as const;

config.env = "prod"; // ❌ Cannot assign to 'env' because it is a read-only property
```

Type of `config` is now:

```ts
{
  readonly env: "dev";
  readonly port: 3000;
}
```

---

## ⚙️ Literal Types + Narrowing

Literal types work seamlessly with **control flow narrowing**:

```ts
type Response = "ok" | "fail";

function handleResponse(r: Response) {
  if (r === "ok") {
    // here, r is narrowed to "ok"
  } else {
    // here, r is narrowed to "fail"
  }
}
```

---

## ⚠️ Common Pitfalls

|Pitfall|Example|Fix|
|---|---|---|
|Type widening with `let`|`let mode = "dark"; // type: string`|Use `const` or `as const`|
|Overly narrow literal|`let val: 10 = 20;`|Assign same literal value|
|Forgetting union|`let type: "car";` (only "car")|Use `"car"|

---

## 🧭 Summary

|Concept|Explanation|Example|
|---|---|---|
|Literal Type|Restricts variable to a specific value|`let x: "yes"|
|Inferred Literal|`const name = "TS";` ⇒ `"TS"`||
|Widened Type|`let name = "TS";` ⇒ `string`||
|`as const`|Makes all fields literal and readonly|`{ role: "user" } as const`|
|Typical Use|Config values, modes, statuses|`"light"|

---

## ✅ Quick Recap

```ts
// String literal union
type Direction = "up" | "down" | "left" | "right";

// Literal inference
const role = "admin"; // type: "admin"

// Preserve literals
const config = { mode: "dark" } as const;

// Narrowing
function log(status: "success" | "error") {
  if (status === "success") console.log("Yay!");
}
```

---

Would you like me to make a **follow-up note on “Literal Inference and `as const` Deep Dive”**, explaining how TS decides whether to widen or preserve literals (with diagrams)? It’s one of the trickiest but most powerful parts of mastering TS types.