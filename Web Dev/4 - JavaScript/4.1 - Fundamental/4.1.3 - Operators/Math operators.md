---
tags:
  - js
  - operator
  - fundamental
---

## 🔹 1. Addition `+`

- Works with numbers **and** strings.
    
- If either operand is a string, it performs **string concatenation** instead of numeric addition.
    

```js
5 + 3;         // 8
"Hello " + "World"; // "Hello World"
"5" + 3;       // "53" (string concatenation!)
```

---

## 🔹 2. Subtraction `-`

- Purely numeric.
    
- If operands are not numbers, JavaScript tries to convert them.
    

```js
10 - 4;    // 6
"10" - 4;  // 6 (string coerced to number)
"hi" - 2;  // NaN (not a number)
```

---

## 🔹 3. Multiplication `*`

- Multiplies numbers.
    
- If one operand is not numeric, JS tries to coerce.
    

```js
6 * 3;     // 18
"6" * 3;   // 18 (string coerced)
"hi" * 2;  // NaN
```

---

## 🔹 4. Division `/`

- Divides left operand by right.
    
- Division by zero → `Infinity` or `-Infinity`.
    
- `0 / 0` gives `NaN`.
    

```js
10 / 2;   // 5
7 / 2;    // 3.5
10 / 0;   // Infinity
0 / 0;    // NaN
```

---

## 🔹 5. Remainder `%`

- Returns the **remainder** after division.
    
- Not the same as "modulus" in math for negatives.
    

```js
10 % 3;   // 1
-10 % 3;  // -1 (sign of the left operand is kept)
```

---

## 🔹 6. Exponentiation `**`

- Raises base to the power of exponent.
    
- Equivalent to `Math.pow()`.
    

```js
2 ** 3;    // 8
5 ** 0;    // 1
(-2) ** 3; // -8
```

---

### 🔹 Operator Precedence (highest → lowest among these)

1. Exponentiation `**`
    
2. Multiplication `*`, Division `/`, Remainder `%`
    
3. Addition `+`, Subtraction `-`
    

So:

```js
2 + 3 * 4 ** 2; // 2 + 3 * 16 = 50
```

---

### 🔹 Type Coercion Rules

- JavaScript tries to **convert non-numeric values** when possible.
    
- Special cases:
    
    ```js
    true + 1;     // 2   (true → 1)
    false + 1;    // 1   (false → 0)
    null + 1;     // 1   (null → 0)
    undefined + 1;// NaN (undefined can’t convert to number)
    ```
    

---

✅ **In short**:  
JavaScript math operators work as expected on numbers, but because of **type coercion** and **special values (`NaN`, `Infinity`)**, you sometimes get surprising results.