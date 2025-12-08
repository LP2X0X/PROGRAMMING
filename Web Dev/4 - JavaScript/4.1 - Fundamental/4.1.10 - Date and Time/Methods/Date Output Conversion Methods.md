---
tags: 
 - js
 - date
 - conversion
 - methods
---

When a JavaScript `Date` object is converted to a string, the output depends on _which_ method you call. These methods differ in:

- timezone (local vs UTC)
    
- format (ISO, readable, locale-aware)
    
- precision (milliseconds or not)
    
- usage (logging, formatting, storage, APIs)
    

Below is the full overview.

---

# **1. `.toString()` — Local Time, Human-Readable**

**Example:**

```js
new Date().toString();
```

**Output sample:**

```
Sun Dec 7 2025 13:44:09 GMT+0700 (Indochina Time)
```

**Characteristics:**

- Uses **local machine timezone**.
    
- Includes:
    
    - day of week
        
    - month name
        
    - date
        
    - time (HH:mm:ss)
        
    - timezone name and offset
        
- Format is **not standardized** between environments (browser vs Node can differ).
    
- Not suitable for storage or APIs.
    

**Use case:**

- Console logs
    
- Debugging
    
- Displaying a quick readable value
    
- NOT for persistence
    

---

# **2. `.toISOString()` — ISO 8601, UTC**

**Example:**

```js
new Date().toISOString();
```

**Output sample:**

```
2025-12-07T06:44:09.195Z
```

**Characteristics:**

- **Always UTC** (Z = Zulu = UTC).
    
- Precise to **milliseconds**.
    
- Stable, strict, predictable.
    
- Standardized by **ISO 8601**.
    
- Ideal for data interchange and persistence.
    

**Use case:**

- Sending timestamps to a backend
    
- Storing in DBs
    
- APIs, JSON, logging systems
    
- Comparing dates reliably
    

---

# **3. `.toUTCString()` — Human-Readable UTC**

**Example:**

```js
new Date().toUTCString();
```

**Sample:**

```
Sun, 07 Dec 2025 06:44:09 GMT
```

**Characteristics:**

- UTC timezone
    
- More readable than ISO
    
- RFC-style format
    
- Less strict than ISO, so not preferred for machine storage
    

**Use case:**

- Displaying UTC to humans
    
- Logging in UTC
    

---

# **4. Locale-Aware Methods**

These format using the user’s locale (language + region).

## **4.1 `.toLocaleString()` — Full Date + Time**

```js
new Date().toLocaleString();
```

Example (Vietnam locale):

```
07/12/2025, 13:44:09
```

**Characteristics:**

- Uses the browser’s locale or one you specify.
    
- Automatically formats numbers, separators, AM/PM, etc.
    
- **Respects timezone** unless overridden.
    

**Use case:**

- Displaying dates/times to end users
    
- UI formatting
    

---

## **4.2 `.toLocaleDateString()` — Date Only**

```js
new Date().toLocaleDateString();
```

Example:

```
07/12/2025
```

**Characteristics:**

- Date only
    
- Locale-specific order (MM/DD/YYYY vs DD/MM/YYYY)
    

Use case:

- UI date labels
    

---

## **4.3 `.toLocaleTimeString()` — Time Only**

```js
new Date().toLocaleTimeString();
```

Example:

```
13:44:09
```

**Characteristics:**

- Time only
    
- Locale rules (24h vs 12h)
    

Use case:

- Showing time in UI
    

---

# **5. Controlled Output with Options**

All locale methods support options:

```js
date.toLocaleString("en-US", {
  weekday: "long",
  year: "numeric",
  month: "short",
  day: "numeric",
  hour: "2-digit",
  minute: "2-digit",
});
```

Example:

```
Sunday, Dec 7, 2025, 01:44 PM
```

**Best for:**

- Full control over formatting without libraries
    
- Localized UI
    

---

# **6. Implicit Conversions**

Some operations automatically convert dates to strings:

### **Concatenation**

```js
new Date() + ""
```

→ Automatically calls `.toString()`.

### **JSON serialization**

```js
JSON.stringify({ date: new Date() })
```

Outputs:

```json
{ "date": "2025-12-07T06:44:09.195Z" }
```

Because JSON uses `.toJSON()` → which internally calls **`.toISOString()`**.

---

# **7. Summary Table**

| Method                 | Timezone             | Format             | Use For            |
| ---------------------- | -------------------- | ------------------ | ------------------ |
| `toString()`           | Local                | Human-readable     | Debugging          |
| `toISOString()`        | UTC                  | ISO 8601           | Storage, APIs      |
| `toUTCString()`        | UTC                  | Human-readable UTC | Logs               |
| `toLocaleString()`     | Local (or specified) | Human-readable     | UI                 |
| `toLocaleDateString()` | Local                | Date               | UI                 |
| `toLocaleTimeString()` | Local                | Time               | UI                 |
| `toJSON()`             | UTC                  | ISO 8601           | JSON serialization |

----

Below is a **complete, structured, and detailed reference** for the **options object** used by:

- `toLocaleString()`
    
- `toLocaleDateString()`
    
- `toLocaleTimeString()`
    
- `Intl.DateTimeFormat`
    

This is the **Intl DateTimeFormat options object**, which controls how JavaScript formats dates and times.

---

# **Intl Date Formatting Options (Complete Detail)**

The options object looks like this:

```js
date.toLocaleString("en-US", {
  optionName: optionValue,
});
```

Each option controls a part of the formatted date.  
You can mix and match as needed.

---

# **1. Date Formatting Options**

These control **year, month, day, weekday**.

### **year**

Format the year:

- `"numeric"` → 2025
    
- `"2-digit"` → 25
    

Example:

```js
year: "numeric"
```

---

### **month**

Format the month in different styles:

- `"numeric"` → 12
    
- `"2-digit"` → 12
    
- `"long"` → December
    
- `"short"` → Dec
    
- `"narrow"` → D
    

Examples:

```js
month: "long"
month: "numeric"
```

---

### **day**

Controls the day of month:

- `"numeric"` → 7
    
- `"2-digit"` → 07
    

---

### **weekday**

Controls day of week:

- `"long"` → Sunday
    
- `"short"` → Sun
    
- `"narrow"` → S
    

---

### **era**

Optional historical calendar info:

- `"long"` → Anno Domini
    
- `"short"` → AD
    
- `"narrow"` → A
    

Rarely used.

---

# **2. Time Formatting Options**

These control **hour, minute, second**, plus AM/PM and timezones.

### **hour**

- `"numeric"` → 1
    
- `"2-digit"` → 01
    

But hour formatting interacts with:

### **hour12**

Force 12h or 24h format:

```js
hour12: true  // 1:44 PM
hour12: false // 13:44
```

If omitted, browser chooses based on locale (e.g., Vietnamese uses 24h).

---

### **minute**

- `"numeric"`
    
- `"2-digit"`
    

---

### **second**

- `"numeric"`
    
- `"2-digit"`
    

---

# **3. Fractional Seconds**

### **fractionalSecondDigits**

Show milliseconds:

```js
fractionalSecondDigits: 1 // .1
fractionalSecondDigits: 2 // .19
fractionalSecondDigits: 3 // .195
```

---

# **4. Time Zone Controls**

### **timeZone**

Convert to a specific timezone:

```js
timeZone: "UTC"
timeZone: "Asia/Ho_Chi_Minh"
timeZone: "America/New_York"
```

If omitted → uses user’s local timezone.

---

### **timeZoneName**

Display timezone label:

- `"long"` → Indochina Time
    
- `"short"` → GMT+7
    
- `"shortOffset"` → GMT+07:00
    
- `"longOffset"` → UTC+07:00
    
- `"shortGeneric"` → ICT
    
- `"longGeneric"` → Indochina Time
    

Example:

```js
timeZoneName: "short"
```

---

# **5. Calendar & Numbering**

### **calendar**

Specify non-Gregorian calendars:

```js
calendar: "buddhist"
calendar: "japanese"
calendar: "islamic"
```

---

### **numberingSystem**

Digit system:

```js
numberingSystem: "latn"  // normal 0–9
numberingSystem: "arab"
numberingSystem: "thai"
```

---

# **6. Formatting Style Shortcuts**

Instead of specifying everything, you can use presets.

### **dateStyle**

- `"full"` → Sunday, December 7, 2025
    
- `"long"` → December 7, 2025
    
- `"medium"` → Dec 7, 2025
    
- `"short"` → 12/7/25
    

Example:

```js
dateStyle: "long"
```

---

### **timeStyle**

- `"full"` → 1:44:09 PM GMT+7
    
- `"long"` → 1:44:09 PM GMT+7
    
- `"medium"` → 1:44:09 PM
    
- `"short"` → 1:44 PM
    

Example:

```js
timeStyle: "short"
```

**Note:** `dateStyle` and `timeStyle` override most individual settings.

---

# **7. Additional Options**

### **localeMatcher**

How locale is chosen:

```js
localeMatcher: "best fit" // default
localeMatcher: "lookup"
```

This is only relevant when you provide multiple fallback locales.

---

# **8. A Complete Example**

```js
const options = {
  weekday: "long",
  year: "numeric",
  month: "long",
  day: "2-digit",
  hour: "2-digit",
  minute: "2-digit",
  second: "2-digit",
  fractionalSecondDigits: 3,
  hour12: false,
  timeZone: "Asia/Ho_Chi_Minh",
  timeZoneName: "short",
};

new Date().toLocaleString("en-US", options);
```

Example output:

```
Sunday, December 07, 2025, 13:44:09.195 GMT+7
```

---

# **Summary Table**

| Option                 | Values                                | Controls             |
| ---------------------- | ------------------------------------- | -------------------- |
| year                   | numeric, 2-digit                      | Year format          |
| month                  | numeric, 2-digit, long, short, narrow | Month                |
| day                    | numeric, 2-digit                      | Day                  |
| weekday                | long, short, narrow                   | Day of week          |
| hour                   | numeric, 2-digit                      | Hour                 |
| minute                 | numeric, 2-digit                      | Minute               |
| second                 | numeric, 2-digit                      | Second               |
| fractionalSecondDigits | 1–3                                   | Milliseconds         |
| hour12                 | true/false                            | 12h vs 24h           |
| timeZone               | IANA strings                          | Timezone             |
| timeZoneName           | long, short, offset                   | TZ label             |
| dateStyle              | full, long, medium, short             | Shortcut date format |
| timeStyle              | full, long, medium, short             | Shortcut time format |
| calendar               | gregory, buddhist, islamic, etc.      | Calendar system      |
| numberingSystem        | latn, arab, thai, etc.                | Number digits        |
| localeMatcher          | lookup, best fit                      | Locale resolution    |
