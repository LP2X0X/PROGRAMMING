---
tags:
  - js
  - term
  - fundamental
---

### **\== (Equality Operator)**
- Compares **values** after **type coercion**.
- If the types are different, JavaScript tries to convert them to the same type before comparing.
- This can lead to some unexpected results.

**Example:**
```js
'5' == 5       // true      → string '5' converted to number 5
false == 0     // true      → false becomes 0
null == undefined // true  → special rule
'0' == false   // true      → '0' → 0, false → 0
[] == ''       // true      → [] → '', then both are ""
[] == 0        // true      → [] → '', then → 0
```

```ad-important
null only equal undefined and vice versa.
```

---
- When JavaScript uses == and needs to do type coercion, it **does not simply coerce only the left or only the right side** — the coercion depends on the types involved, and it can coerce one or both operands based on specific rules.
	- Coercion happens depending on the operand types.
	- Often the string or boolean is converted to number.
	- Sometimes object is converted to primitive.
	- It’s not always just left or right — it’s about the types involved and JS internal rules.