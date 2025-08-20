---
tags: 
 - js 
 - function
 - technique
---

#### Requirement

- `debounce(f, ms)` returns a wrapper that delays calling `f` until there’s been no new calls for `ms` milliseconds. Only the latest call’s arguments are used.
- Example: with `f = debounce(f, 1000)`, if calls happen at 0ms, 200ms, and 500ms, the actual `f` runs **once** at 1500ms (1000ms after the last call).
![[Pasted image 20250820143000.png|center|500]]

---
#### Solution

```js
function debounce(f, ms) {
	let timeoutId;                                             // null here
	return function (...args) {                               // Spread it to get passed in arguments instead of using "arguments" built in variable
		clearTimeout(timeoutId);                               // clearTimeout can be called on null
		timeoutId = setTimeout(() => f.apply(this, args), ms); // setTimeout with new call to reset the timer
	}
}
```