---
tags:
  - js
  - date
---

- The _autocorrection_ is a very handy feature of `Date` objects. We can set out-of-range values, and it will auto-adjust itself.
	```js
	let date = new Date(2013, 0, 32); // 32 Jan 2013 ?!?
	alert(date); // ...is 1st Feb 2013!
	```

- Let’s say we need to increase the date “28 Feb 2016” by 2 days. It may be “2 Mar” or “1 Mar” in case of a leap-year. We don’t need to think about it. Just add 2 days. The `Date` object will do the rest:
	```js
	let date = new Date(2016, 1, 28);
	date.setDate(date.getDate() + 2);
	
	alert( date ); // 1 Mar 2016
	```

- That feature is often used to get the date after the given period of time. For instance, let’s get the date for “70 seconds after now”:
	```javascript
	let date = new Date();
	date.setSeconds(date.getSeconds() + 70);
	
	alert( date ); // shows the correct date
	```

- We can also set zero or even negative values. For example:
	```js
	let date = new Date(2016, 0, 2); // 2 Jan 2016
	
	date.setDate(1); // set day 1 of month
	alert( date );
	
	date.setDate(0); // min day is 1, so the last day of the previous month is assumed
	alert( date ); // 31 Dec 2015
	```