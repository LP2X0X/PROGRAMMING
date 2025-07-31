---
tags: js, date, access
---

- There are methods to access the year, month and so on from the `Date` object:
	- [getFullYear()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getFullYear) Get the year (4 digits)
	- [getMonth()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getMonth) Get the month, **from 0 to 11**.
	- [getDate()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getDate) Get the day of month, from 1 to 31, the name of the method does look a little bit strange.
	- [getHours()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getHours), [getMinutes()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getMinutes), [getSeconds()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getSeconds), [getMilliseconds()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getMilliseconds) Get the corresponding time components.
	- [getDay()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getDay) Get the day of week, **from `0` (Sunday) to `6` (Saturday)**. The first day is always Sunday, in some countries that’s not so, but can’t be changed.
	- [getTime()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime) Returns the timestamp for the date – a number of milliseconds passed from the January 1st of 1970 UTC+0. ^11faa2
	- [getTimezoneOffset()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getTimezoneOffset) Returns the difference between UTC and the local time zone, in minutes.
