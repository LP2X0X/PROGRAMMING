---
tags: 
 - js
 - error
 - term
 - object
---

- When an error occurs, JavaScript generates an object containing the details about it. The object is then passed as an argument to `catch`:
	```js
	try {
	  // ...
	} catch (err) { // <-- the "error object", could use another word instead of err
	  // ...
	}
	```

- For all built-in errors, the error object has two main properties:
	- `name`: Error name. For instance, for an undefined variable that’s `"ReferenceError"`.
	- `message`: Textual message about error details.
	- There are other non-standard properties available in most environments. One of most widely used and supported is: `stack`. It represent current call stack: a string with information about the sequence of nested calls that led to the error. Used for debugging purposes.
	```js
	try {
	  lalala; // error, variable is not defined!
	} catch (err) {
	  alert(err.name); // ReferenceError
	  alert(err.message); // lalala is not defined
	  alert(err.stack); // ReferenceError: lalala is not defined at (...call stack)
	
	  // Can also show an error as a whole
	  // The error is converted to string as "name: message"
	  alert(err); // ReferenceError: lalala is not defined
	}
	```