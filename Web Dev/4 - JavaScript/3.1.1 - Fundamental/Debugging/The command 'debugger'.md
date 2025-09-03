---
tags:
  - js
  - debug
  - fundamental
---

- We can also pause the code by using the `debugger` command in it, like this:
	```js
	function hello(name) {
	  let phrase = `Hello, ${name}!`;
	
	  debugger;  // <-- the debugger stops here
	
	  say(phrase);
	}
	```

- Such command works only when the development tools are open, otherwise the browser ignores it.