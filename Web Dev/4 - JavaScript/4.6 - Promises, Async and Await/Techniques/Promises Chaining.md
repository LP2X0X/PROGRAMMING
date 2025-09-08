---
tags: 
 - js
 - technique
 - advance
---

```js
new Promise(function(resolve, reject) {

  setTimeout(() => resolve(1), 1000); // (*)

}).then(function(result) { // (**)

  alert(result); // 1
  return result * 2;

}).then(function(result) { // (***)

  alert(result); // 2
  return result * 2;

}).then(function(result) {

  alert(result); // 4
  return result * 2;

});
```

- The idea is that the result is passed through the chain of `.then` handlers. Here the flow is:
	1. The initial promise resolves in 1 second `(*)`,
	2. Then the `.then` handler is called `(**)`, which in turn creates a new promise (resolved with `2` value).
	3. The next `then` `(***)` gets the result of the previous one, processes it (doubles) and passes it to the next handler.
	4. …and so on.

- As the result is passed along the chain of handlers, we can see a sequence of `alert` calls: `1` → `2` → `4`.
![[Pasted image 20250819102159.png|center|200]]

```ad-note
The whole thing works, **because every call to a `.then` returns a new promise**, so that we can call the next `.then` on it.
When a handler returns a value, it becomes the result of that promise, so the next `.then` is called with it.
```

---

### **A classic newbie error**
- Technically we can also add many `.then` to a single promise. This is not chaining.
	```js
	let promise = new Promise(function(resolve, reject) {
	  setTimeout(() => resolve(1), 1000);
	});
	
	promise.then(function(result) {
	  alert(result); // 1
	  return result * 2;
	});
	
	promise.then(function(result) {
	  alert(result); // 1
	  return result * 2;
	});
	
	promise.then(function(result) {
	  alert(result); // 1
	  return result * 2;
	});
	```
- All `.then` on the same promise get the same result – the result of that promise. So in the code above all `alert` show the same: `1`.
</br>
- What we did here is just adding several handlers to one promise. They don’t pass the result to each other; instead they process it independently.
![[Pasted image 20250819102224.png|center]]