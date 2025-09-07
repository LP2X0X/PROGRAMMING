---
tags: 
 - js
 - function
 - technique
---

- What if we’d like to fix some arguments, but not the context `this`? For example, for an object method.

```js
function partial (func, ...argsBound){
	return function(...args){
		return func.call(this, ...argsBound, ...args);
	}
}
```

----

### Example of usage

```js
// Usage:
let user = {
  firstName: "John",
  say(time, phrase) {
    alert(`[${time}] ${this.firstName}: ${phrase}!`);
  }
};

// add a partial method with fixed time
user.sayNow = partial(user.say, new Date().getHours() + ':' + new Date().getMinutes());

user.sayNow("Hello");
// Something like:
// [10:00] John: Hello!
```