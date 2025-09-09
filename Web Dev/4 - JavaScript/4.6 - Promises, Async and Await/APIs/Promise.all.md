---
tags: 
 - js
 - promise
 - api
---

- Let’s say we want many promises to execute in parallel and wait until all of them are ready. For instance, download several URLs in parallel and process the content once they are all done. That’s what `Promise.all` is for.

- The syntax is:
	```js
	let promise = Promise.all(iterable);
	```

- `Promise.all` takes an iterable (usually, an array of promises) and *returns a new promise*. The new promise resolves when all listed promises are resolved, and the array of their results becomes its result.

- For instance, the `Promise.all` below settles after 3 seconds, and then its result is an array `[1, 2, 3]`:
	```js
	Promise.all([
	  new Promise(resolve => setTimeout(() => resolve(1), 3000)), // 1
	  new Promise(resolve => setTimeout(() => resolve(2), 2000)), // 2
	  new Promise(resolve => setTimeout(() => resolve(3), 1000))  // 3
	]).then(alert); // 1,2,3 when promises are ready: each promise contributes an array member
	```

```ad-note
Please note that the order of the resulting array members is the same as in its source promises. Even though the first promise takes the longest time to resolve, it’s still first in the array of results.
```

- A common trick is to map an array of job data into an array of promises, and then wrap that into `Promise.all`. For instance, if we have an array of URLs, we can fetch them all like this:
	```js
	let urls = [
	  'https://api.github.com/users/iliakan',
	  'https://api.github.com/users/remy',
	  'https://api.github.com/users/jeresig'
	];
	
	// map every url to the promise of the fetch
	let requests = urls.map(url => fetch(url));
	
	// Promise.all waits until all jobs are resolved
	Promise.all(requests)
	  .then(responses => responses.forEach(
	    response => alert(`${response.url}: ${response.status}`)
	  ));
	```

- A bigger example with fetching user information for an array of GitHub users by their names (we could fetch an array of goods by their ids, the logic is identical):
	```js
	let names = ['iliakan', 'remy', 'jeresig'];
	
	let requests = names.map(name => fetch(`https://api.github.com/users/${name}`));
	
	Promise.all(requests)
	  .then(responses => {
	    // all responses are resolved successfully
	    for(let response of responses) {
	      alert(`${response.url}: ${response.status}`); // shows 200 for every url
	    }
	
	    return responses;
	  })
	  // map array of responses into an array of response.json() to read their content
	  .then(responses => Promise.all(responses.map(r => r.json())))
	  // all JSON answers are parsed: "users" is the array of them
	  .then(users => users.forEach(user => alert(user.name)));
	```

- **If any of the promises is rejected, the promise returned by `Promise.all` immediately rejects with that error.**
	```js
	Promise.all([
	  new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
	  new Promise((resolve, reject) => setTimeout(() => reject(new Error("Whoops!")), 2000)),
	  new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
	]).catch(alert); // Error: Whoops!
	```

```ad-note
This means that you **cannot get the rest of the results** when one fails.
```

- Here the second promise rejects in two seconds. That leads to an immediate rejection of `Promise.all`, so `.catch` executes: the rejection error becomes the outcome of the entire `Promise.all`.

```ad-note
#### In case of an error, other promises are ignored

If one promise rejects, `Promise.all` immediately rejects, completely forgetting about the other ones in the list. Their results are ignored.

For example, if there are multiple `fetch` calls, like in the example above, and one fails, the others will still continue to execute, but `Promise.all` won’t watch them anymore. They will probably settle, but their results will be ignored.

`Promise.all` does nothing to cancel them, as there’s no concept of “cancellation” in promises.
```

````ad-note

#### `Promise.all(iterable)` allows non-promise “regular” values in `iterable`

Normally, `Promise.all(...)` accepts an iterable (in most cases an array) of promises. But if any of those objects is not a promise, it’s passed to the resulting array “as is”.

For instance, here the results are `[1, 2, 3]`:

```js
Promise.all([
  new Promise((resolve, reject) => {
    setTimeout(() => resolve(1), 1000)
  }),
  2,
  3
]).then(alert); // 1, 2, 3
```

So we are able to pass ready values to `Promise.all` where convenient.
````
