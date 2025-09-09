---
tags: 
 - js
 - note
---

Both `await` and `.then` are ways to “wait” for a promise to settle — the difference is **how the waiting looks in code** and **where the continuation goes**.

---

## 🔹 `.then` version

```js
fetch("/data.json")
  .then(resp => resp.json())
  .then(data => {
    console.log(data); // runs when the promise fulfills
  });
console.log("after .then");
```

- `console.log("after .then")` runs **immediately**, before the fetch finishes.
    
- The continuation (`data => { ... }`) is scheduled as a **microtask** once the promise resolves.
    

---

## 🔹 `await` version

```js
async function load() {
  let resp = await fetch("/data.json");
  let data = await resp.json();
  console.log(data); // runs when promise fulfills
}
load();
console.log("after load");
```

- When JS hits `await fetch(...)`, it **pauses only this async function**.
    
- The rest of the program (`console.log("after load")`) continues right away.
    
- When the promise fulfills, execution of `load()` resumes **as if it were synchronous code**.
    

---

## 🔹 Key comparison

|Feature|`.then`|`await`|
|---|---|---|
|Style|Callback-based (chained)|Looks synchronous|
|Where code continues|Inside `.then(...)` handler|After the `await` line|
|Whole program paused?|❌ No|❌ No (only that async function is suspended)|
|Microtask involvement|Yes (handler queued as microtask)|Yes (resumes via microtask too)|

---

✅ So your statement is **spot on**:  
👉 `await` is just syntactic sugar for `.then`, except it lets you write “wait here until settled” instead of nesting callbacks.