---
tags: js, constructor, function, fundamental
---

- Constructor functions technically are regular functions. There are two conventions though:
	1. They are named with capital letter first.
	2. They should be executed only with `"new"` operator.
```js
function User(name) {
  this.name = name;
  this.isAdmin = false;
}

let user = new User("Jack");

alert(user.name); // Jack
alert(user.isAdmin); // false
```

- When a function is executed with `new`, it does the following steps:
	1. A new empty object is created and assigned to `this`.
	2. The function body executes. Usually it modifies `this`, adds new properties to it.
	3. The value of `this` is returned.
```js
function User(name) {
  // this = {};  (implicitly)

  // add properties to this
  this.name = name;
  this.isAdmin = false;

  // return this;  (implicitly)
}
```

- Of course, we can add to `this` not only properties, but methods as well
```js
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert( "My name is: " + this.name );
  };
}

let john = new User("John");

john.sayHi(); // My name is: John

/*
john = {
   name: "John",
   sayHi: function() { ... }
}
*/
```