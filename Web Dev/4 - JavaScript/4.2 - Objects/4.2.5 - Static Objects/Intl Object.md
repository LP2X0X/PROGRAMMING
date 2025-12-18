---
tags: 
 - js
 - object
 - static
 - special
---

## 1. The Core Concept

The `Intl` object is the namespace for the ECMAScript Internationalization API. It provides language-sensitive string comparison, number formatting, and date and time formatting.

Why use it?

Instead of hardcoding formats (e.g., assuming dates are MM/DD/YYYY or decimals use dots 1.50), Intl offloads this complexity to the browser, which has a massive database of cultural patterns.

### The Universal Syntax

Almost every `Intl` constructor follows this pattern:

```js
new Intl.Constructor(locales, options).format(value);
```

1. **`locales`**: A string (BCP 47 language tag) like `'en-US'`, `'de-DE'`, `'ja-JP'`. Passing `undefined` uses the user's browser default.
    
2. **`options`**: An object configuring the output (style, currency, digits, etc.).
    

---

## 2. Formatting Numbers (`Intl.NumberFormat`)

Used for currencies, percentages, decimals, and units.

### A. Currency

The safest way to handle money. It handles symbols (`$`, `€`, `¥`) and their placement (prefix vs. suffix) automatically.

```js
const money = 123456.789;

// US Dollar
console.log(new Intl.NumberFormat('en-US', { 
    style: 'currency', 
    currency: 'USD' 
}).format(money)); 
// Output: "$123,456.79" (Rounds automatically)

// German Euro (uses comma for decimals, dot for thousands)
console.log(new Intl.NumberFormat('de-DE', { 
    style: 'currency', 
    currency: 'EUR' 
}).format(money)); 
// Output: "123.456,79 €"
```

### B. Units (Engineering/Science)

Handles spacing and pluralization of units automatically.

```js
const speed = 50;

console.log(new Intl.NumberFormat('en-US', {
    style: 'unit',
    unit: 'mile-per-hour'
}).format(speed));
// Output: "50 mph"

console.log(new Intl.NumberFormat('en-US', {
    style: 'unit',
    unit: 'liter',
    unitDisplay: 'long' // 'narrow', 'short', 'long'
}).format(speed));
// Output: "50 liters"
```

### C. Notation (Compact/Scientific)

Great for dashboards with large numbers (Likes, Views).

```js
const views = 1500000;

console.log(new Intl.NumberFormat('en-US', {
    notation: 'compact',
    compactDisplay: 'short'
}).format(views));
// Output: "1.5M"
```

---

## 3. Formatting Dates & Times (`Intl.DateTimeFormat`)

A lightweight alternative to libraries like Moment.js or date-fns for _formatting_.

### A. Shortcuts (`dateStyle` / `timeStyle`)

The easiest way to get standard formats.

```js
const now = new Date();
const fmt = new Intl.DateTimeFormat('en-GB', {
    dateStyle: 'full', 
    timeStyle: 'short' 
});

console.log(fmt.format(now));
// Output: "Sunday, 7 December 2025 at 13:30"
```

### B. Granular Control

If you need specific parts.

```js
const fmt = new Intl.DateTimeFormat('en-US', {
    day: 'numeric',
    month: 'short', // "Dec"
    year: '2-digit', // "25"
    weekday: 'long', // "Sunday"
    hour12: false // 24-hour clock
});
```

---

## 4. Relative Time (`Intl.RelativeTimeFormat`)

Formats distinct points in time relative to "now" (e.g., "yesterday", "in 5 minutes").

- **Options:** `numeric: 'auto'` allows strings like "yesterday". `numeric: 'always'` forces "1 day ago".
    

```js
const rtf = new Intl.RelativeTimeFormat('en-US', { numeric: 'auto' });

console.log(rtf.format(-1, 'day')); // "yesterday"
console.log(rtf.format(-2, 'day')); // "2 days ago"
console.log(rtf.format(10, 'minute')); // "in 10 minutes"
```

---

## 5. Lists (`Intl.ListFormat`)

Joins arrays into a natural language string. Handles the oxford comma and "and/or" logic.

```js
const vehicles = ['Motorcycle', 'Bus', 'Car'];

const formatter = new Intl.ListFormat('en-US', { 
    style: 'long', 
    type: 'conjunction' // 'conjunction' (and), 'disjunction' (or), 'unit' (list)
});

console.log(formatter.format(vehicles));
// Output: "Motorcycle, Bus, and Car"
```

---

## 6. Sorting & Comparison (`Intl.Collator`)

**Crucial:** The standard `array.sort()` sorts by ASCII code, which fails for accents (e.g., 'z' comes before 'ä' in ASCII, but in German 'ä' is near 'a').

```js
const names = ['Zebra', 'Äpfel', 'Banana'];

// Bad (Standard Sort)
console.log([...names].sort()); 
// Output: ['Banana', 'Zebra', 'Äpfel'] (Accents often pushed to end)

// Good (Intl Collator)
console.log([...names].sort(new Intl.Collator('de').compare));
// Output: ['Äpfel', 'Banana', 'Zebra']
```

---

## 7. Pluralization (`Intl.PluralRules`)

Different languages have different rules for when a word changes form. English only has "one" and "other" (plural). Some languages have rules for "few", "many", "two", etc.

This API returns the **category**, not the string.

```js
const rules = new Intl.PluralRules('en-US');

console.log(rules.select(0)); // "other" (0 items)
console.log(rules.select(1)); // "one"  (1 item)
console.log(rules.select(2)); // "other" (2 items)

// Russian has complex rules
const ruRules = new Intl.PluralRules('ru-RU');
console.log(ruRules.select(4)); // "few"
console.log(ruRules.select(5)); // "many"
```

---

## 8. Translating Names (`Intl.DisplayNames`)

Converts region/language codes into human-readable strings.

```js
const regionNames = new Intl.DisplayNames(['en-US'], { type: 'region' });

console.log(regionNames.of('US')); // "United States"
console.log(regionNames.of('VN')); // "Vietnam"
console.log(regionNames.of('DE')); // "Germany"
```

---

## 9. Performance & Optimization ⚡

This is the most important technical takeaway.

The Problem:

Creating a new Intl.NumberFormat instance is computationally expensive. If you do it inside a loop, it will kill performance.

**The Bad Way:**

```js
// Creates 1000 objects in memory
arr.forEach(num => {
   console.log(num.toLocaleString('en-US')); // Internally creates an Intl object
});
```

The Good Way:

Create the formatter once, then reuse the .format method.

```js
const dollarFmt = new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' });

arr.forEach(num => {
   console.log(dollarFmt.format(num)); // Fast!
});
```