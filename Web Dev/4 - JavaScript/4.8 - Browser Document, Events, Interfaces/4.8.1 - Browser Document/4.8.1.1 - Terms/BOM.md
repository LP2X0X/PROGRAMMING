---
tags: 
 - js
 - BOM
 - term
---

### 🧩 What It Is

The **BOM (Browser Object Model)** allows JavaScript to **interact with the browser itself**, not the web page content. It represents additional objects provided by the browser (host environment) for working with everything except the document.

It provides objects that represent the **browser window, history, screen, location, navigation**, and more.

> 💡 The BOM is **not standardized** like the DOM — it can vary slightly between browsers.

---

### 🏗️ BOM Hierarchy Overview

```
window
 ├── document      → The web page (DOM)
 ├── location      → URL info
 ├── history       → Browser history
 ├── navigator     → Browser info
 ├── screen        → Display info
 ├── console       → Debugging output
 └── localStorage  → Persistent storage
```

🪟 Everything starts from the **`window` object**, which is the **global object** in the browser.

---

### 🪟 1️⃣ `window` Object

- Represents the **browser window/tab**
    
- All global variables and functions become its properties.
    

```js
console.log(window.innerWidth, window.innerHeight); // viewport size
window.open("https://example.com"); // open new tab/window
window.alert("Hello!"); // show alert
window.setTimeout(() => console.log("Done"), 1000);
```

🔹 Common Properties

|Property|Description|
|---|---|
|`innerWidth`, `innerHeight`|Viewport size|
|`outerWidth`, `outerHeight`|Full browser window|
|`screenX`, `screenY`|Window position on screen|
|`scrollX`, `scrollY`|Scroll offsets|

🔹 Common Methods

|Method|Description|
|---|---|
|`alert()`, `confirm()`, `prompt()`|Dialogs|
|`open(url)`|Open new tab|
|`close()`|Close current window|
|`setTimeout(fn, ms)`|Delay|
|`setInterval(fn, ms)`|Repeat|

---

### 📍 2️⃣ `location` Object

Provides info about the **current URL** and methods to navigate.

```js
console.log(location.href);      // full URL
location.href = "https://google.com"; // redirect
location.reload();               // reload page
```

|Property|Example|Description|
|---|---|---|
|`href`|`https://example.com/path?x=1`|Full URL|
|`protocol`|`https:`|Protocol|
|`hostname`|`example.com`|Domain|
|`pathname`|`/path`|Path after domain|
|`search`|`?x=1`|Query string|
|`hash`|`#section`|Fragment|
|`reload()`|–|Reload page|
|`assign(url)`|–|Navigate to URL|
|`replace(url)`|–|Navigate w/o history entry|

---

### ⏪ 3️⃣ `history` Object

Controls the **browser’s session history**.

```js
history.back();   // Go to previous page
history.forward(); // Go to next page
history.go(-2);    // Go 2 pages back
```

|Property|Description|
|---|---|
|`length`|Number of history entries|
|`state`|Current state object|

|Method|Description|
|---|---|
|`back()`|Previous page|
|`forward()`|Next page|
|`go(n)`|Relative navigation|
|`pushState(state, title, url)`|Add entry|
|`replaceState(state, title, url)`|Replace current entry|

💡 `pushState()` and `replaceState()` are used in **Single Page Applications (SPAs)**.

---

### 🧭 4️⃣ `navigator` Object

Contains information about the **browser and operating system**.

```js
console.log(navigator.userAgent);
console.log(navigator.language);
```

|Property|Example|Description|
|---|---|---|
|`userAgent`|`Mozilla/5.0 ...`|Browser info string|
|`language`|`en-US`|Current language|
|`platform`|`Win32`|OS info|
|`onLine`|`true`|Internet connection status|
|`geolocation`|Object|Access device location|

Example:

```js
navigator.geolocation.getCurrentPosition(pos => {
  console.log(pos.coords.latitude, pos.coords.longitude);
});
```

---

### 🖥️ 5️⃣ `screen` Object

Gives info about the **user’s display**.

```js
console.log(screen.width, screen.height);
console.log(screen.availWidth, screen.availHeight);
```

|Property|Description|
|---|---|
|`width`, `height`|Total screen size|
|`availWidth`, `availHeight`|Usable area excluding taskbar|
|`colorDepth`|Bit depth per pixel|

---

### 💾 6️⃣ Storage APIs (Part of BOM)

Store data directly in the browser.

```js
// Local storage (persistent)
localStorage.setItem("user", "Long");
console.log(localStorage.getItem("user"));

// Session storage (tab-based)
sessionStorage.setItem("theme", "dark");
```

|Storage|Lifespan|Shared Between Tabs|
|---|---|---|
|`localStorage`|Permanent (until cleared)|✅ Yes|
|`sessionStorage`|Until tab closed|❌ No|

---

### 🧰 7️⃣ `console` Object

For debugging output.

```js
console.log("Hello");
console.error("Error!");
console.table([{id:1}, {id:2}]);
```

|Method|Description|
|---|---|
|`log()`|Normal output|
|`warn()`|Warning|
|`error()`|Error message|
|`table()`|Tabular data|
|`time()` / `timeEnd()`|Timing performance|

---

### 🔗 BOM vs DOM

|Feature|BOM|DOM|
|---|---|---|
|Purpose|Browser control|Page content control|
|Root object|`window`|`document`|
|Examples|`location`, `history`, `screen`|`getElementById()`, `querySelector()`|
|Standardized?|❌ Not fully|✅ Yes (W3C)|

---

### 🧠 Quick Recap

- `window` → everything
    
- `location` → URL control
    
- `history` → navigation stack
    
- `navigator` → browser/device info
    
- `screen` → display metrics
    
- `storage` → persistent data
    
- `console` → debugging helper
    