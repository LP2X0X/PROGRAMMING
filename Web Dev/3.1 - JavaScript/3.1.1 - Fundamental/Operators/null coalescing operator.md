---
tags:
  - js
  - operator
  - fundamental
---

- The nullish coalescing operator `??` returns the first argument if it’s not `null/undefined`.
- The result of `a ?? b` is:
	- If `a` is defined, then `a`,
	- If `a` isn’t defined, then `b`.

---

### Usage
#### 1 - Provide default value
```javascript
let user;

alert(user ?? "Anonymous"); // Anonymous (user is undefined)
```

#### 2 - Select the first value which is not null \ undefined
```javascript
let firstName = null;
let lastName = null;
let nickName = "Supercoder";

// shows the first defined value:
alert(firstName ?? lastName ?? nickName ?? "Anonymous"); // Supercoder
```

---

```ad-warning
Due to safety reasons, JavaScript forbids using `??` together with `&&` and `||` operators, **unless the precedence is explicitly specified with parentheses**.
```