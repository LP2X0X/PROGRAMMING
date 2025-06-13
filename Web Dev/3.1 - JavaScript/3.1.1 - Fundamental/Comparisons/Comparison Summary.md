---
tags: js, comparison, fundamental, summary
---

### 📌 Equality Comparison (== vs \=\==)

#### 🟢 [[Strict Equality Operator|Strict Equality]] (=== and !\=\=)

- Compares values and types (no conversion).
- Safer and more predictable.
- Example:
	```js
	5 === '5' → false  
	null === undefined → false
	```

#### 🟡 [[Equality Operator|Loose Equality]] (== and !=)

- Compares values after type conversion ("coercion").
- Can cause confusing results.
- Example:
	```js
	 '5' == 5 → true  
	 null == undefined → true  
	 [] == '' → true  
	 [1] == '1' → true
	```

```ad-note
- Recommendation:
	✅ Prefer === over == for most comparisons.
	✅ Use == only when checking for null or undefined (if needed).
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
### 📌 Relational Comparison (<, >, <=, >=)

#### 🟠 Rules:

- If both are strings → compared alphabetically (lexical order).
- If types differ → both converted to numbers (after calling toPrimitive if needed).
- If result is NaN → comparison returns false.

- Examples:

```js
'5' < '12' → false (because '5' > '1' in string)
'5' < 12 → true → '5' → 5 → 5 < 12
[] < 1 → true → [] → '' → 0 < 1
[1, 2] > '1' → false → '1,2' → NaN > 1 → false
NaN < 5 → false
```


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
### 📌 Special Cases to Watch

Expression               | Result | Why
-------------------------|--------|---------------------
null == undefined        | true   | Special case
NaN == NaN               | false  | NaN not equal to anything
0 == false               | true   | false → 0
[] == ''                 | true   | [] → '' (toString)
[1] == '1'               | true   | [1].toString() → '1'
{} == '[object Object]'  | false  | Object compared to string

```ad-note
Use Object.is(x, y) → safer for edge cases like NaN or +0 vs -0.
```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  
### ✅ Best Practices

- Use \=\== and !\=\= for equality.
- Be careful with < and > when comparing strings and objects.
- Avoid == unless you understand how coercion works.
- Use Object.is(x, y) to compare NaN or distinguish +0 and -0.