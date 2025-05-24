---
tags: js, term, fundamental
---

- A template literal **is a special kind of string** that can evaluate any expressions embedded within it.
- This gives you the flexibility to dynamically populate a string with the values of variables, the results of calculations, or other code, instead of having to type out every character of the string exactly or combine several variables into a string with the + operator.
  
---

### How to create
- Template literals are enclosed in backticks (\`) instead of quotation marks.
- You incorporate code using *placeholder syntax*, which look like this: ${ }.
```js
let name = "Nick";
'Hello ${name}!';
-> `Hello Nick!`
'There are ${60 * 60 * 24} seconds in a day';
-> `There are 86400 seconds in a day`
```
