---
tags: 
 - js
 - promise
 - api
---

- Similar to `Promise.all`, but waits only for the first settled promise and gets its result (or error).
- The syntax is:
	```js
	let promise = Promise.race(iterable);
	```

- For instance, here the result will be `1`:
	```js
	Promise.race([
	  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
	  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
	  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
	]).then(alert); // 1
	```

- The first promise here was fastest, so it became the result. After the first settled promise “wins the race”, all further results/errors are ignored.