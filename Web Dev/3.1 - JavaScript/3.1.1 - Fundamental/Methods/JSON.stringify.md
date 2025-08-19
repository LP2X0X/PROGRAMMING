---
tags:
  - js
  - json
  - method
---

- The JSON.stringify method converts a JavaScript object into a JSON string. The resulting `json` string is called a _JSON-encoded_ or _serialized_ or _stringified_ or _marshalled_ object.
	```js
	let json = JSON.stringify(nestedObject)
	'{"name":"Outer","content":{"name":"Middle","content":{"name":"Inner","content":"Whoa…"}}}
	```

- Example:
	```js
	let student = {
	  name: 'John',
	  age: 30,
	  isAdmin: false,
	  courses: ['html', 'css', 'js'],
	  spouse: null
	};
	
	let json = JSON.stringify(student);
	
	alert(typeof json); // we've got a string!
	
	alert(json);
	/* JSON-encoded object:
	{
	  "name": "John",
	  "age": 30,
	  "isAdmin": false,
	  "courses": ["html", "css", "js"],
	  "spouse": null
	}
	*/
	```

---

### Syntax explanation
- The full syntax of `JSON.stringify` is:
	```javascript
	let json = JSON.stringify(value[, replacer, space])
	```
	- **value**: A value to encode.
	- **replacer**: *Array of properties* to encode or a *mapping function*: function(key, value).
		```js
		let room = {
		  number: 23
		};
		
		let meetup = {
		  title: "Conference",
		  participants: [{name: "John"}, {name: "Alice"}],
		  place: room // meetup references room
		};
		
		room.occupiedBy = meetup; // room references meetup
		
		alert( JSON.stringify(meetup, ['title', 'participants', 'place', 'name', 'number']) );
		/*
		{
		  "title":"Conference",
		  "participants":[{"name":"John"},{"name":"Alice"}],
		  "place":{"number":23}
		}
		*/
		```
		```js
		let room = {
		  number: 23
		};
		
		let meetup = {
		  title: "Conference",
		  participants: [{name: "John"}, {name: "Alice"}],
		  place: room // meetup references room
		};
		
		room.occupiedBy = meetup; // room references meetup
		
		alert( JSON.stringify(meetup, function replacer(key, value) {
		  alert(`${key}: ${value}`);
		  return (key == 'occupiedBy') ? undefined : value;
		}));
		
		/* key:value pairs that come to replacer:
		:             [object Object]
		title:        Conference
		participants: [object Object],[object Object]
		0:            [object Object]
		name:         John
		1:            [object Object]
		name:         Alice
		place:        [object Object]
		number:       23
		occupiedBy: [object Object]
		*/
		```
	- **space** Amount of space to use for formatting.
---

### Limitation

- JSON is data-only language-independent specification, so some JavaScript-specific object properties are skipped by `JSON.stringify`. Namely:
	- Function properties (methods).
	- Symbolic keys and values.
	- Properties that store `undefined`.
	```javascript
	let user = {
	  sayHi() { // ignored
	    alert("Hello");
	  },
	  [Symbol("id")]: 123, // ignored
	  something: undefined // ignored
	};
	
	alert( JSON.stringify(user) ); // {} (empty object)
	```

- The important limitation: there must be no circular references. For instance:
	```js
	let room = {
	  number: 23
	};
	
	let meetup = {
	  title: "Conference",
	  participants: ["john", "ann"]
	};
	
	meetup.place = room;       // meetup references room
	room.occupiedBy = meetup; // room references meetup
	
	JSON.stringify(meetup); // Error: Converting circular structure to JSON
	```

---

- All that’s missing from this representation are the original line breaks and indentations we used to clarify the object literal’s nested structure. To re-create those, we can pass JSON.stringify another argument representing the number of spaces to indent each new nested object:
	```js
	JSON.stringify(value, replacer, space)
	```

|**Parameter**|**Description**|
|---|---|
|value|Object or value to stringify|
|replacer (optional)|Function or array to filter properties|
|space    (optional)|Number or string for pretty printing|