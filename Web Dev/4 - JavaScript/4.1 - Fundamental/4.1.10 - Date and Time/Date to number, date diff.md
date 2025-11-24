---
tags:
  - js
  - date
  - feature
---

- When a `Date` object is converted to number, it becomes the timestamp same as [[Get Components#^11faa2|date.getTime()]]:
	```js
	let date = new Date();
	alert(+date); // the number of milliseconds, same as date.getTime()
	```
- The important side effect: dates can be subtracted, the result is their difference in ms. That can be used for time measurements:
	```js
	let start = new Date(); // start measuring time
	
	// do the job
	for (let i = 0; i < 100000; i++) {
	  let doSomething = i * i * i;
	}
	
	let end = new Date(); // end measuring time
	
	alert( `The loop took ${end - start} ms` );
	```