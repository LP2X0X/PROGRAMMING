---
tags:
  - js
  - destructuring
  - object
  - fundamental
---

- The basic syntax is:
	```js
	let {var1, var2} = {var1:…, var2:…}
	```
- We should have an existing object on the right side, that we want to split into variables. The left side contains an object-like “pattern” for corresponding properties. In the simplest case, that’s a list of variable names in `{...}`.
	```js
	let options = {
	  title: "Menu",
	  width: 100,
	  height: 200
	};
	
	let {title, width, height} = options;
	
	alert(title);  // Menu
	alert(width);  // 100
	alert(height); // 200
	```

- Properties `options.title`, `options.width` and `options.height` are assigned to the corresponding variables.
````ad-note
The order does not matter. This works too:
```js
// changed the order in let {...}
let {height, width, title} = { title: "Menu", height: 200, width: 100 }
````

---

### Custom name

- If we want to assign a property to a variable with another name, for instance, make `options.width` go into the variable named `w`, then we can set the variable name using a colon:
	```js
	let options = {
	  title: "Menu",
	  width: 100,
	  height: 200
	};
	
	// { sourceProperty: targetVariable }
	let {width: w, height: h, title} = options;
	
	// width -> w
	// height -> h
	// title -> title
	
	alert(title);  // Menu
	alert(w);      // 100
	alert(h);      // 200
	```
- The colon shows “what : goes where”. In the example above the property `width` goes to `w`, property `height` goes to `h`, and `title` is assigned to the same name.

---

### Default value

- For potentially missing properties we can set default values using `"="`, like this:
	```js
	let options = {
	  title: "Menu"
	};
	
	let {width = 100, height = 200, title} = options;
	
	alert(title);  // Menu
	alert(width);  // 100
	alert(height); // 200
	```
- Just like with arrays or function parameters, default values can be any expressions or even function calls. They will be evaluated **if the value is not provided**.
- In the code below `prompt` asks for `width`, but not for `title`:
	```js
	let options = {
	  title: "Menu"
	};
	
	let {width = prompt("width?"), title = prompt("title?")} = options;
	
	alert(title);  // Menu
	alert(width);  // (whatever the result of prompt is)
	```