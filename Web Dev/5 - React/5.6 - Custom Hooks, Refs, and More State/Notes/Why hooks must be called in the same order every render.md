---
tags: 
 - js
 - hook 
 - note
---

# Why Hooks Must Be Called in the Same Order

## 1. How React Stores Hooks

* On the initial render, React builds a **fiber tree**, where each fiber represents a component.
* Each fiber holds:

  * Props
  * Work to do
  * And importantly → a **linked list of hooks** used in that component.

This linked list connects hooks **in the exact order they were called** during the render.

![[Pasted image 20250909191159.png|center|500]]

---

## 2. Why Order Matters

* React does not recreate fibers (and their hook lists) from scratch on each render.
* Instead, it **reuses the same list** and updates it.
* Therefore, React assumes:

  * Hook #1 (first call) → always the same hook between renders
  * Hook #2 (second call) → always the same hook between renders
  * …and so on

This mapping (hook position ↔ hook state) is how React knows which state or effect belongs to which hook.

---

## 3. What Breaks If Order Changes

If a hook is called conditionally:

```js
if (condition) {
  useState("B"); // might run only sometimes ❌
}
useEffect(...);
```

* On the first render, the `useState("B")` is hook #2.
* On the next render (if the condition is false), hook #2 disappears.
* The linked list is now broken:

  * React still expects a second hook there.
  * The `useEffect` shifts into the wrong slot.
  * State and effects get mismatched → React becomes confused.

---

## 4. Rule of Hooks

To avoid this problem:

* **Hooks must always be called at the top level of a component.**
* Never inside conditionals, loops, or nested functions.
* This guarantees that the **order of hook calls is the same on every render**.

---

## 5. Why React Uses This Design

- Well, the big reason is that a linked list, which relies on the hook call order, is the simplest way to associate each hook with its value. So basically, the order in which the hook is called uniquely identifies the hook. For example, React knows that hook number one has a state of 23, and that hook number two has a state of empty string. So the value is associated with the call order. And this is very convenient because by using the call order, we developers don't have to manually assign names to each hook, which would create multiple problems that we don't need to get into.

* Using call order as an identifier is a **simple and efficient** way to track hooks.
* Developers don’t need to name or register hooks manually.
* Without this rule, React would need a more complex system (slower, more error-prone).

---

✅ **In short:**
Hooks rely on **call order** to link state/effects with the right hook across renders. Changing the order breaks this mapping, so React enforces the rule: **always call hooks in the same order, at the top level of your component.**

![[Pasted image 20250909193506.png|center]]