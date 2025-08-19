---
tags:
  - js
  - term
  - scope
  - fundamental
---

- If a variable is declared inside a code block `{...}`, it’s only visible inside that block.
	```js
	{
	  // do some job with local variables that should not be seen outside
	
	  let message = "Hello"; // only visible in this block
	
	  alert(message); // Hello
	}
	
	alert(message); // Error: message is not defined
	```

- We can use this to isolate a piece of code that does its own task, with variables that only belong to it:
	```js
	{
	  // show message
	  let message = "Hello";
	  alert(message);
	}
	
	{
	  // show another message
	  let message = "Goodbye";
	  alert(message);
	}
	```