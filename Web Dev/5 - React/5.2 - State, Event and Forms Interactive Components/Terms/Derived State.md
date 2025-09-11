---
tags:
  - react
  - term
  - fundamental
---

In React (and many other UI frameworks), **derived state** refers to **state that can be calculated from other state or props**, rather than being stored directly.

```ad-note
Derived state is updated on each re-render.
```

---

## **🧠 What Is Derived State?**

It’s **not managed directly** using useState() — instead, it’s computed **on the fly** from other values.

### **✅ Example:**

```jsx
function Cart({ items }) {
  // Derived state: total price
  const total = items.reduce((sum, item) => sum + item.price, 0);

  return <div>Total: ${total}</div>;
}
```

- Here, total is **derived** from items. There’s **no need to store it in state** because:
	- It doesn’t need to be preserved across renders
	- It always reflects the latest value of items

---

## **❌ Bad Example (Don’t Do This)**

```jsx
const [total, setTotal] = useState(0);

useEffect(() => {
  setTotal(items.reduce((sum, item) => sum + item.price, 0));
}, [items]);
```

This is **unnecessary** and makes the code more complex and error-prone. You should only store total if you’re modifying it independently.

---

## **🛠 When Should You Use Derived State?**

|**Use derived state (computed) if…**|**Use** useState **if…**|
|---|---|
|It depends only on props or other state|The value is independently controlled|
|It should always reflect the latest data|The value changes over time manually|
|You can compute it efficiently|You need to persist or update it arbitrarily|

---

## **✅ Another Example: Filtered List**

```jsx
function PackingList({ items }) {
  const packedItems = items.filter(item => item.packed);
  const unpackedItems = items.filter(item => !item.packed);

  return (
    <>
      <h2>Packed</h2>
      <ul>{packedItems.map(item => <li key={item.id}>{item.description}</li>)}</ul>

      <h2>Unpacked</h2>
      <ul>{unpackedItems.map(item => <li key={item.id}>{item.description}</li>)}</ul>
    </>
  );
}
```

Here, packedItems and unpackedItems are derived state — they don’t need to be stored.

---

### **TL;DR:**

- **Derived state = computed on the fly**
- **Avoid duplicating state** that can be computed
- Keeps components simpler and avoids bugs