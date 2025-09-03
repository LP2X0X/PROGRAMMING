---
tags:
  - js
  - note
---

- When a function is passed in `setInterval/setTimeout`, an internal reference is created to it and saved in the scheduler. It prevents the function from being garbage collected, even if there are no other references to it.
	```js
	// the function stays in memory until the scheduler calls it
	setTimeout(function() {...}, 100);
	```

- For `setInterval` the function stays in memory until `clearInterval` is called.
- There’s a side effect. A function references the outer lexical environment, so, while it lives, outer variables live too. They may take much more memory than the function itself. 

```ad-important
When we don’t need the scheduled function anymore, it’s better to cancel it, even if it’s very small.
```