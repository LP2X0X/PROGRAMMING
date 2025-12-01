---
tags: 
 - js
 - overview
 - browser
 - environment
---

![[Pasted image 20251028150716.png|center|600]]

### 🪟 `window` — The Root

- The **`window`** object is at the **top** — representing the **global scope** in browsers.
    
- Everything (DOM, BOM, and JavaScript built-ins) is **accessible through `window`**.
    

---

### 🌳 DOM (Document Object Model)

- Represented on the **left**:
    
    - `document` is a property of `window`, giving access to the **web page structure** (HTML).
        
    - Belongs to the **DOM**, not BOM.
        

---

### 🌐 BOM (Browser Object Model)

- In the **middle column**, all are **part of BOM**:
    
    - `navigator` 🧭 – browser info
        
    - `screen` 💻 – display info
        
    - `location` 📍 – current URL
        
    - `frames` 🪟 – subframes within the window
        
    - `history` ⏳ – session history
        
    - `XMLHttpRequest` 📡 – network requests (older than `fetch`)
        

These are **browser-specific objects** provided by the environment, not by the JavaScript language itself.

---

### 💻 JavaScript Core Objects

- On the **right**:
    
    - `Object`, `Array`, `Function`, etc. are **built-in language objects** — part of ECMAScript.
        
    - They come from **JavaScript**, not the browser.
        

---

### ✅ Summary

|Part|Provided By|Examples|Purpose|
|---|---|---|---|
|**DOM**|Browser (via `document`)|`document`, `element`, `querySelector()`|Interact with webpage structure|
|**BOM**|Browser (via `window`)|`navigator`, `location`, `history`|Interact with browser environment|
|**JavaScript Core**|ECMAScript language|`Object`, `Array`, `Function`|Logic, data, computation|