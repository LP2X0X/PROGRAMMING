---
tags: 
 - js
 - date
 - object
 - conversion
---

# **📌 Overview of Conversion in the JavaScript `Date` Object**

The JavaScript `Date` object automatically performs **multiple types of conversions**, depending on how you use it. Understanding these conversions helps avoid bugs, especially with time zones and string parsing.

Below is the full breakdown.

---

# **1️⃣ Internal Representation (How Date Stores Time)**

- A `Date` object always stores time as a **single number**:
    
    **milliseconds since January 1, 1970, UTC (Unix epoch)**
    

Example:

```js
const d = new Date();
console.log(d.getTime());
```

✔ Returns an integer → internal timestamp  
❌ Not human-readable  
❌ Not tied to any timezone

This is **universal** across all systems.

---

# **2️⃣ Input Conversion (Creating Dates)**

When you create a `Date`, JavaScript converts the input into a timestamp.

## **A. ISO String → Interpreted as UTC**

```js
new Date("2023-10-12");
```

Interpreted as **UTC midnight**, then converted to local time for display.

## **B. Date + Time String**

```js
new Date("2023-10-12T10:00:00Z"); // Z = UTC
```

## **C. Without "Z"**

```js
new Date("2023-10-12T10:00:00");
```

Interpreted as **local time**.

## **D. Numeric Arguments**

```js
new Date(2023, 0, 1); // Jan 1, 2023 at local time 00:00
```

⚠ Month is 0-based  
✔ Local timezone is used

## **E. Using timestamp**

```js
new Date(1700000000000);
```

✔ This bypasses parsing  
✔ Safest way

---

# **3️⃣ Output Conversion (Displaying Dates)**

When you convert a date into a string or print it, JavaScript converts the timestamp into another format.

## **A. Default `.toString()` (Local Timezone)**

```js
new Date().toString();
```

Converts timestamp → **local time string**  
Includes timezone offset.

## **B. `.toISOString()` (UTC)**

```js
new Date().toISOString();
```

Converts to **ISO 8601** string in UTC.

## **C. Localized Formats**

```js
new Date().toLocaleString();
new Date().toLocaleDateString();
new Date().toLocaleTimeString();
```

Converts to a locale-aware format (based on browser locale).

---

# **4️⃣ Conversion When Comparing or Using Math**

When a `Date` is used in numeric operations, JavaScript automatically converts it to its timestamp.

## **A. Date → Number**

```js
+new Date();     // timestamp
new Date() - 0;  // timestamp
```

## **B. Date subtraction**

```js
new Date("2020") - new Date("2019");
```

✔ Returns **difference in milliseconds**  
No need to call `.getTime()` manually.

---

# **5️⃣ JSON Conversion**

When converting to JSON, dates become ISO strings:

```js
JSON.stringify({ created: new Date() });
```

→ `{"created":"2025-11-15T12:00:00.000Z"}`

---

# **6️⃣ Timezone Conversion**

JavaScript stores time in **UTC**, but displays it in **local time** unless you use UTC methods.

## **Local Time Methods**

```
getHours()
getFullYear()
getMonth()
```

## **UTC Time Methods**

```
getUTCHours()
getUTCFullYear()
getUTCMonth()
```

---

# **7️⃣ Type Conversion Rules (Important)**

JavaScript uses **internal abstract operations** when converting a `Date`:

### **A. Date → Primitive**

Happens when:

- Comparing
    
- Using `+`
    
- Using template strings
    

JavaScript tries in this order:

1. `.valueOf()` → timestamp
    
2. `.toString()` → date string
    

Example:

```js
"" + new Date(); // string
new Date() + 1;  // number
```

---

# **8️⃣ Automatic Parsing Caveats**

⚠ **String parsing is unreliable unless using ISO format**

Bad:

```js
new Date("12/10/2022");
```

Different browsers interpret differently.

Good:

```js
new Date("2022-12-10"); // ISO
```

---

# **✔ Summary Table**

|Operation|Conversion|Result|
|---|---|---|
|Create Date|Input → timestamp|UTC timestamp|
|Print Date|timestamp → string|Local or UTC string|
|Math|Date → number|Timestamp|
|JSON|Date → ISO string|Universal|
|Compare|Date → number|Timestamp|
|Methods|timestamp → local UTC parts|Dates/hours/minutes|