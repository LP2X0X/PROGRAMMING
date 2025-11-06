---
tags:
  - js
  - note
---

**Object coercion** refers to the process where JavaScript converts an object into a primitive value (`string`, `number`, or `boolean`) when an operation requires it.

## Conversion rules

1. There’s no conversion to boolean. All objects are `true` in a boolean context, as simple as that. There exist only numeric and string conversions.
2. The numeric conversion happens when we subtract objects or apply mathematical functions. For instance, `Date` objects can be subtracted, and the result of `date1 - date2` is the time difference between two dates.
3. As for the string conversion – it usually happens when we output an object with `alert(obj)` and in similar contexts

- We can implement string and numeric conversion by ourselves, using special object methods.

---

## Hints

How does JavaScript decide which conversion to apply?
There are three variants of type conversion, that happen in various situations. They’re called “hints”.

```ad-note
You can think of a _hint_ as the context the JavaScript engine uses to decide which type it should convert the object into.
```

### `"string"`

For an object-to-string conversion, when we’re doing an operation on an object that expects a string, like `alert`:

```javascript
// output
alert(obj);

// using object as a property key
anotherObj[obj] = 123;
```

### `"number"`

For an object-to-number conversion, like when we’re doing maths:

```javascript
// explicit conversion
let num = Number(obj);

// maths (except binary plus)
let n = +obj; // unary plus
let delta = date1 - date2;

// less/greater comparison
let greater = user1 > user2;
```

Most built-in mathematical functions also include such conversion.

### `"default"`

Occurs in rare cases when the operator is “not sure” what type to expect.

For instance, binary plus `+` can work both with strings (concatenates them) and numbers (adds them). So if a binary plus gets an object as an argument, it uses the `"default"` hint to convert it.

Also, if an object is compared using `==` with a string, number or a symbol, it’s also unclear which conversion should be done, so the `"default"` hint is used.

```javascript
// binary plus uses the "default" hint
let total = obj1 + obj2;

// obj == number uses the "default" hint
if (user == 1) { ... };
```

The greater and less comparison operators, such as `<` `>`, can work with both strings and numbers too. Still, they use the `"number"` hint, not `"default"`. That’s for historical reasons.

---

**To do the conversion, JavaScript tries to find and call three object methods:**

1. Call `obj[Symbol.toPrimitive](hint)` – the method with the symbolic key `Symbol.toPrimitive` (system symbol), if such method exists,
2. Otherwise if hint is `"string"`
    - try calling `obj.toString()` or `obj.valueOf()`, whatever exists.
3. Otherwise if hint is `"number"` or `"default"`
    - try calling `obj.valueOf()` or `obj.toString()`, whatever exists.

### Symbol.toPrimitive

- There’s a built-in [[Symbol|symbol]] named `Symbol.toPrimitive` that should be used to name the conversion method, like this:

```javascript
obj[Symbol.toPrimitive] = function(hint) {
  // here goes the code to convert this object to a primitive
  // it must return a primitive value
  // hint = one of "string", "number", "default"
};
```

If the method `Symbol.toPrimitive` exists, it’s used for all hints, and no more methods are needed.

For instance, here `user` object implements it:

```javascript
let user = {
  name: "John",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// conversions demo:
alert(user); // hint: string -> {name: "John"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

As we can see from the code, `user` becomes a self-descriptive string or a money amount, depending on the conversion. The single method `user[Symbol.toPrimitive]` handles all conversion cases.

### toString / valueOf

- If there’s no `Symbol.toPrimitive` then JavaScript tries to find methods `toString` and `valueOf`:
	- For the `"string"` hint: call `toString` method, and if it doesn’t exist or if it returns an object instead of a primitive value, then call `valueOf` (so `toString` has the priority for string conversions).
	- For other hints: call `valueOf`, and if it doesn’t exist or if it returns an object instead of a primitive value, then call `toString` (so `valueOf` has the priority for maths).

Methods `toString` and `valueOf` come from ancient times. They are not symbols (symbols did not exist that long ago), but rather “regular” string-named methods. They provide an alternative “old-style” way to implement the conversion.

These methods must return a primitive value. If `toString` or `valueOf` returns an object, then it’s ignored (same as if there were no method).

- By default, a plain object has following `toString` and `valueOf` methods:
	- The `toString` method returns a string `"[object Object]"`.
	- The `valueOf` method returns the object itself.

```js
let user = {
  name: "John",
  money: 1000,

  // for hint="string"
  toString() {
    return `{name: "${this.name}"}`;
  },

  // for hint="number" or "default"
  valueOf() {
    return this.money;
  }

};

alert(user); // toString -> {name: "John"}
alert(+user); // valueOf -> 1000
alert(user + 500); // valueOf -> 1500
```

````ad-note
In the absence of `Symbol.toPrimitive` and `valueOf`, `toString` will handle all primitive conversions.
```js
let user = {
  name: "John",

  toString() {
    return this.name;
  }
};

alert(user); // toString -> John
alert(user + 500); // toString -> John500
```
````