---
tags:
  - js
  - date
  - create
---

## 1. Creating Dates

### Without arguments → current date & time

```js
let now = new Date();
console.log(now); // current date/time
```

### From timestamp (ms since Jan 1, 1970 UTC+0)

```js
let Jan01_1970 = new Date(0);                  // Thu Jan 01 1970
let Jan02_1970 = new Date(24 * 3600 * 1000);   // Fri Jan 02 1970
```

### From string (parsed automatically)

```js
let date = new Date("2017-01-26");
console.log(date);
```

⚠️ Parsed date strings are **interpreted as UTC** and then adjusted to your local timezone.

### From components

```js
new Date(year, month, day?, hours?, minutes?, seconds?, ms?)
```

- `year`: 4 digits (e.g., `2025`).
    
    - If 2 digits, treated as `19xx` (`98` → `1998`).
        
- `month`: **0–11** (`0 = January`, `11 = December`).
    
- `day`: defaults to `1`.
    
- `hours/minutes/seconds/ms`: default to `0`.
    
- `0` for day = last day of previous month.
    

```js
new Date(2011, 0, 1);                // Jan 1, 2011, 00:00:00
new Date(2011, 0, 1, 2, 3, 4, 567);  // Jan 1, 2011, 02:03:04.567
```

---

## 2. Current timestamp

```js
let ms = Date.now(); // milliseconds since 1970-01-01 UTC
```

Handy for measuring performance:

```js
let start = Date.now();
// ...do stuff...
let end = Date.now();
console.log(`Took ${end - start} ms`);
```

---

## 3. Parsing dates

```js
Date.parse("YYYY-MM-DDTHH:mm:ss.sssZ");
```

- Example:
    
    ```js
    let ms = Date.parse("2012-01-26T13:51:50.417-07:00");
    console.log(ms); // timestamp in ms
    let date = new Date(ms);
    console.log(date);
    ```
    
- Format:
    
    - `YYYY-MM-DD` (date part, required at minimum).
        
    - `"T"` → separator.
        
    - `HH:mm:ss.sss` (time part).
        
    - Optional timezone:
        
        - `Z` → UTC.
            
        - `±hh:mm` → offset.
            

---

## 4. Important notes

- `Date` objects always represent an **exact timestamp**, but getters/setters can show it in local time or UTC.
    
- Months are **0-based** (easy to forget).
    
- Day of week starts at **0 = Sunday**.
    
- Precision is **1 ms**.
    
- Arithmetic works with timestamps:
    
    ```js
    let tomorrow = new Date(Date.now() + 24*3600*1000);
    ```
    

---

✅ With this, you’ve got a **practical summary + gotchas** for working with dates in JS.