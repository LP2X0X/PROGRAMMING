---
tags: 
 - js
 - promise
 - api
---

- Similar to `Promise.race`, but waits only for the first fulfilled promise and gets its result. If all of the given promises are rejected, then the returned promise is rejected with [`AggregateError`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AggregateError) – a special error object that stores all promise errors in its `errors` property.

- The syntax is:
	```js
	let promise = Promise.any(iterable);
	```

- For instance, here the result will be `1`:
	```js
	Promise.any([
	  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 1000)),
	  new Promise((resolve, reject) => setTimeout(() => resolve(1), 2000)),
	  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
	]).then(alert); // 1
	```

- The first promise here was fastest, but it was rejected, so the second promise became the result. After the first fulfilled promise “wins the race”, all further results are ignored.

- Here’s an example when all promises fail:
	```js
	Promise.any([
	  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Ouch!")), 1000)),
	  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Error!")), 2000))
	]).catch(error => {
	  console.log(error.constructor.name); // AggregateError
	  console.log(error.errors[0]); // Error: Ouch!
	  console.log(error.errors[1]); // Error: Error!
	});
	```

- As you can see, error objects for failed promises are available in the `errors` property of the `AggregateError` object.