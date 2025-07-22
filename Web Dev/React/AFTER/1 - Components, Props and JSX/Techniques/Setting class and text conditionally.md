---
tags: react, technique, fundamental
---

To **conditionally set a class and text in React**, you can use **ternary operators**, **logical operators**, or **template strings** inside JSX.

Here’s how you can do it:

---

## **✅ 1. Conditional Text**

```jsx
function Status({ isOnline }) {
  return <p>{isOnline ? "Online 🟢" : "Offline 🔴"}</p>;
}
```

- isOnline is a boolean.
- It changes the text based on its value.

---

## **✅ 2. Conditional Class Name**

```jsx
function Status({ isOnline }) {
  return (
    <p className={isOnline ? "status-online" : "status-offline"}>
      {isOnline ? "Online" : "Offline"}
    </p>
  );
}
```

- You can toggle CSS class based on a condition.
    

---

## **✅ 3. Add Class Conditionally (on top of base class)**

```jsx
function Status({ isOnline }) {
  return (
    <p className={`status ${isOnline ? "online" : "offline"}`}>
      {isOnline ? "Online" : "Offline"}
    </p>
  );
}
```

```ad-note
Template literal is a JS expression. Therefore we need to enter JS mode using curly brackets.
```

---

## **✅ 4.** 

## **Using Logical AND (&&) for Short Conditions**

```
<p className={isError && "error"}>Something went wrong</p>
```

- If isError is false → className={false} → no class applied.
    

---

## **✅ 5. Tailwind CSS Style Example**

If you’re using Tailwind:

```jsx
<button className={`btn ${isLoading ? "opacity-50" : "hover:bg-blue-600"}`}>
  {isLoading ? "Loading..." : "Submit"}
</button>
```

---

## **✅ Bonus: Extract Logic to a Variable**

```jsx
const statusText = isOnline ? "Online" : "Offline";
const statusClass = isOnline ? "status-online" : "status-offline";

return <p className={statusClass}>{statusText}</p>;
```

Helps keep JSX clean.