---
tags: js, comparison, fundamental
---

- **When comparing values of different types using comparison operators (`<`, `>`, etc.), JavaScript usually converts both operands to numbers**. But there’s one big exception: **if both are strings**, then it performs a **string (lexicographical) comparison** instead.
```ad-note
Comparison in this case excluding (or not mean) ***equality*** check.
```

Here’s a **clear table** showing how JavaScript converts various types to **numbers** when using **comparison operators** (`<`, `>`, `<=`, `>=`), based on the **ECMAScript ToNumber coercion rules**:

---

## 🔢 JavaScript Type Conversion to Number (in comparisons)

|**Type**|**Example**|**Converted to Number**|**Notes**|
|---|---|---|---|
|`Number`|`42`|`42`|Already a number|
|`String`|`'123'`|`123`|Parsed as number if numeric, else `NaN`|
|`String`|`'abc'`|`NaN`|Non-numeric string → `NaN`|
|`Boolean`|`true`|`1`|`true → 1`, `false → 0`|
|`null`|`null`|`0`|Special rule|
|`undefined`|`undefined`|`NaN`|Always becomes `NaN`|
|`NaN`|`NaN`|`NaN`|Comparisons with `NaN` are always `false`|
|`[]` (empty array)|`[]`|`0`|`[] → '' → 0`|
|`[1]` (single item)|`[1]`|`1`|`[1] → '1' → 1`|
|`[1,2]` (multi-item array)|`[1,2]`|`NaN`|`[1,2] → '1,2' → NaN`|
|`{}` (object)|`{}`|`NaN`|Can't convert to primitive → `NaN`|
|Custom object with `valueOf`|`{ valueOf: () => 5 }`|`5`|Uses `valueOf` or `toString`|
|`Symbol`|`Symbol('x')`|❌ Error|Throws `TypeError` — cannot convert to number|

---

## 🧠 Summary:

- ✅ Most types **attempt conversion to number** using `ToNumber()`.
- ❗ If conversion results in `NaN`, **any comparison will be `false`**.
- ❌ `Symbol` cannot be converted and throws an error.