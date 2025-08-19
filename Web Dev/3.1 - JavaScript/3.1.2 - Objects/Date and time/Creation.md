---
tags:
  - js
  - date
  - create
---

- To create a new `Date` object call `new Date()` with one of the following arguments:
	- Without arguments – create a `Date` object for the current date and time:
		```js
		let now = new Date();
		alert( now ); // shows current date/time
		```
	- Create a `Date` object with the time equal to number of milliseconds (1/1000 of a second) passed after the Jan 1st of 1970 UTC+0.
		```js
		// 0 means 01.01.1970 UTC+0
		let Jan01_1970 = new Date(0);
		alert( Jan01_1970 );
		
		// now add 24 hours, get 02.01.1970 UTC+0
		let Jan02_1970 = new Date(24 * 3600 * 1000);
		alert( Jan02_1970 );
		```
	- If there is a single argument, and it’s a string, then it is parsed automatically. The algorithm is the same as `Date.parse` uses, we’ll cover it later.
		```js
		let date = new Date("2017-01-26");
		alert(date);
		// The time is not set, so it's assumed to be midnight GMT and
		// is adjusted according to the timezone the code is run in
		// So the result could be
		// Thu Jan 26 2017 11:00:00 GMT+1100 (Australian Eastern Daylight Time)
		// or
		// Wed Jan 25 2017 16:00:00 GMT-0800 (Pacific Standard Time)
		```
	- With `new Date(year, month, date, hours, minutes, seconds, ms)` we can create the date with the given components in the local time zone. Only the first two arguments are obligatory.
		- The `year` should have 4 digits. For compatibility, 2 digits are also accepted and considered `19xx`, e.g. `98` is the same as `1998` here, but always using 4 digits is strongly encouraged.
		- The `month` count starts with `0` (Jan), up to `11` (Dec).
		- The `date` parameter is actually the day of month, if absent then `1` is assumed.
			```ad-tip
			When we pass 0, then it means “one day before 1st day of the month”, in other words: “the last day of the previous month”.
			```
		- If `hours/minutes/seconds/ms` is absent, they are assumed to be equal `0`.
			```javascript
			new Date(2011, 0, 1, 0, 0, 0, 0); // 1 Jan 2011, 00:00:00
			new Date(2011, 0, 1); // the same, hours etc are 0 by default
			```
		- The maximal precision is 1 ms (1/1000 sec):
			```js
			let date = new Date(2011, 0, 1, 2, 3, 4, 567);
			alert( date ); // 1.01.2011, 02:03:04.567
			```
	- Date.now() returns the current timestamp in milliseconds forom 1 Jan 1970:
		```js
		let start = Date.now(); // milliseconds count from 1 Jan 1970
		
		// do the job
		for (let i = 0; i < 100000; i++) {
		  let doSomething = i * i * i;
		}
		
		let end = Date.now(); // done
		
		alert( `The loop took ${end - start} ms` ); // subtract numbers, not dates
		```
	- Parse date from a string:
		- The method [Date.parse(str)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/parse) can read a date from a string.
		- The string format should be: `YYYY-MM-DDTHH:mm:ss.sssZ`, where:
			- `YYYY-MM-DD` – is the date: year-month-day.
			- The character `"T"` is used as the delimiter.
			- `HH:mm:ss.sss` – is the time: hours, minutes, seconds and milliseconds.
			- The optional `'Z'` part denotes the time zone in the format `+-hh:mm`. A single letter `Z` would mean UTC+0.
		- Shorter variants are also possible, like `YYYY-MM-DD` or `YYYY-MM` or even `YYYY`.
		- The call to `Date.parse(str)` parses the string in the given format and returns the timestamp (number of milliseconds from 1 Jan 1970 UTC+0). If the format is invalid, returns `NaN`.
			```js
			let ms = Date.parse('2012-01-26T13:51:50.417-07:00');
			
			alert(ms); // 1327611110417  (timestamp)
			```
		- We can instantly create a `new Date` object from the timestamp:
			```js
			let date = new Date( Date.parse('2012-01-26T13:51:50.417-07:00') );
			
			alert(date);
			```
