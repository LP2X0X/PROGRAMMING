---
tags: 
 - typescript
 - type
---

## ⭐ 1. What is the `Date` type in TypeScript?

In TypeScript, `Date` is **a built-in object type** that represents a point in time.

It is _not_ special to TS — it’s the same JavaScript `Date` object, but with TypeScript type safety.

```ts
let today: Date = new Date();
```

TypeScript simply ensures the value is a valid `Date`.

---

## ⭐ 2. Ways to create a Date

```ts
new Date();                      // current date & time
new Date(2025, 10, 19);          // year, month (0-based!), day
new Date("2025-11-19");          // ISO string
new Date(1731992000000);         // timestamp in ms
```

---

## ⭐ 3. Common Date methods you will use

### **Get parts**

```ts
date.getFullYear();
date.getMonth();        // 0 = January
date.getDate();
date.getHours();
date.getMinutes();
date.getSeconds();
```

### **Set parts**

```ts
date.setFullYear(2026);
date.setMonth(0);       // January
date.setDate(15);       // 15th
```

### **Convert to strings**

```ts
date.toISOString();     // "2025-11-19T08:25:11.345Z"
date.toLocaleString();  // formatted for user's locale
```

### **Get timestamp**

```ts
date.getTime();         // milliseconds since 1970
Date.now();             // shortcut
```

---

## ⭐ 4. Narrowing and checking Dates in TS

When you get data from APIs or JSON, dates come as **strings**, not Date objects.

Example:

```ts
const user = await fetch("/user").then(r => r.json());
console.log(typeof user.birthdate); // "string"
```

To convert:

```ts
const birthdate: Date = new Date(user.birthdate);
```

To check if a value is actually a `Date` object:

```ts
function isDate(value: unknown): value is Date {
  return value instanceof Date && !isNaN(value.getTime());
}
```

---

## ⭐ 5. Regular real-world use cases

## ✔ 1. **Storing timestamps**

```ts
const createdAt: Date = new Date();
```

## ✔ 2. **Parsing dates from API responses**

```ts
interface User {
  id: string;
  createdAt: string; // from server
}

function normalize(user: User) {
  return {
    ...user,
    createdAt: new Date(user.createdAt),
  };
}
```

## ✔ 3. **Comparing dates**

```ts
if (start.getTime() > end.getTime()) {
  throw new Error("Start must be before end");
}
```

## ✔ 4. **Adding time**

```ts
const tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);
```

## ✔ 5. **Formatting for UI**

```ts
const date = new Date();
const formatted = date.toLocaleDateString("en-US", {
  month: "short",
  day: "numeric",
  year: "numeric",
});
```

## ✔ 6. **Working with expiry times**

JWTs, OTP codes, sessions, cookies, etc.

```ts
const expiresAt = new Date(Date.now() + 10 * 60 * 1000); // 10 minutes
```

## ✔ 7. **Sorting date arrays**

```ts
dates.sort((a, b) => a.getTime() - b.getTime());
```

---

# ⭐ 6. Best practices

#### ✔ Store in backend as ISO strings
#### ✔ Convert to `Date` immediately on frontend
#### ✔ Compare using timestamps, not strings
#### ✔ Use `toISOString()` when sending to server
#### ✔ Remember months are 0-based in constructor

```ts
new Date(2025, 0, 1); // January 1, not February
```

---

## ⭐ 7. When NOT to use JS Date

Use libraries if you need:

- Time zones
    
- Durations (e.g., addMonths safely)
    
- Calendar math
    
- Localization
    

Examples:  
`date-fns`, `Day.js`, `Luxon`