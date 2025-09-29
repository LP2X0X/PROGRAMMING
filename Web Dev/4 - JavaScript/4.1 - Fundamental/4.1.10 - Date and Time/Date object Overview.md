---
tags:
  - js
  - date
  - overview
---


# 📅 JavaScript `Date` Object — Overview

## 1. What it is

- `Date` is a built-in JavaScript object for **representing points in time**.
    
- Internally, a date is stored as the **number of milliseconds since Jan 1, 1970, UTC** (the Unix epoch).
    
- It can show the time in **local time zone** or **UTC**.
    

---

## 2. Creating dates

You create a date with `new Date(...)`. Main ways:

1. **Current date/time**
    
    ```js
    let now = new Date();
    ```
    
2. **From timestamp (ms since 1970-01-01 UTC)**
    
    ```js
    let d = new Date(0); // Jan 1, 1970
    ```
    
3. **From string (parsed like Date.parse)**
    
    ```js
    let d = new Date("2017-01-26"); // parses ISO-style string
    ```
    
4. **From components**
    
    ```js
    let d = new Date(2025, 0, 14, 10, 30, 0, 0);
    // year=2025, month=0=Jan, day=14, 10:30 AM
    ```
    

---

## 3. Getting values

- `getFullYear()` → year (4 digits)
    
- `getMonth()` → month (0–11)
    
- `getDate()` → day of month (1–31)
    
- `getDay()` → day of week (0=Sun, …, 6=Sat)
    
- `getHours(), getMinutes(), getSeconds(), getMilliseconds()`
    
- `getTime()` → timestamp in ms
    

👉 Each has a UTC version, e.g. `getUTCFullYear()`.

---

## 4. Setting values

- `setFullYear(year, month?, date?)`
    
- `setMonth(month, date?)`
    
- `setDate(dayOfMonth)`
    
- `setHours(h, m?, s?, ms?)`
    
- `setMinutes(m, s?, ms?)`
    
- `setSeconds(s, ms?)`
    
- `setMilliseconds(ms)`
    
- `setTime(timestamp)`
    

---

## 5. Utilities

- `Date.now()` → current timestamp in ms (faster than `new Date().getTime()`).
    
- `Date.parse(str)` → parse an ISO-format string into a timestamp.
    
    ```js
    Date.parse("2012-01-26T13:51:50.417-07:00");
    ```
    

---

## 6. Important details

- **Months are zero-based**: January = `0`, December = `11`.
    
- **Days of week start with 0 = Sunday**.
    
- **Precision** is 1 millisecond.
    
- `Date` always represents an **absolute point in time** → but getters/setters can be in local time or UTC.
    
- Arithmetic works on timestamps:
    
    ```js
    let tomorrow = new Date(Date.now() + 24*60*60*1000);
    ```
    

---

✅ **In short**:  
`Date` in JavaScript is your tool for working with time. You can create, parse, get, set, and manipulate timestamps — always based on milliseconds since 1970, with methods for both **local** and **UTC** time.