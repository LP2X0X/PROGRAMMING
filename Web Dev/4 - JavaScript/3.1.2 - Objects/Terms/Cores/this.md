---
tags: 
 - js 
 - term 
 - advance
---

### **Using `this` in Methods**

**To access the object, a method can use the `this` keyword.**
The value of `this` is evaluated during the run-time, depending on the context.

```ad-important
The rule is simple: if `obj.f()` is called, then `this` is `obj` during the call of `f`.
```

```js
const user = {
  name: "John",
  sayHi() {
    console.log(this.name);
  }
};

user.sayHi(); // John

```

```ad-note
If you come from another programming language, then you are probably used to the idea of a “bound `this`”, where methods defined in an object always have `this` referencing that object.

In JavaScript `this` is “free”, its value is evaluated at call-time and does not depend on where the method was declared, but rather on what object is “before the dot”.
```

````ad-note
We can even call the function without an object at all:
```js
function sayHi() {
  alert(this);
}

sayHi(); // undefined
```
In this case `this` is `undefined` in strict mode. If we try to access `this.name`, there will be an error.

In non-strict mode the value of `this` in such case will be the _global object_ (`window` in a browser). This is a historical behavior that `"use strict"` fixes.
````

- In summary, The value of `this` is defined at run-time:
	- When a function is declared, it may use `this`, but that `this` has no value until the function is called.
	- A function can be copied between objects.
	- When a function is called in the “method” syntax: `object.method()`, the value of `this` during the call is `object`.

---

### **Arrow functions have no `this`**
- Arrow functions are special: they don’t have their “own” `this`. If we reference `this` from such a function, it’s taken from the **outer “normal” function**.
```js
let user = {
  firstName: "Ilya",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // Ilya
```

----

# What `this` is (core idea)

- `this` is **not** bound at define time (except for arrow functions).
    
- In normal functions, `this` is determined **by how the function is called** (_the call-site_).
    
- In arrow functions, `this` is **lexically captured** from the surrounding scope (no own `this`).
    

# Binding rules (with precedence)

If a value for `this` can come from multiple places, this is the effective order:

1. **`new` binding (constructor call)**
    
    ```js
    function C() { this.x = 1; }
    const o = new C(); // this = the new object
    ```
    
    - If the constructor returns an object explicitly, that object is the result (and `this` is discarded).
        
    - If it returns a primitive, it’s ignored; the new instance is returned.
        
2. **Explicit binding**: `call`, `apply`, `bind`
    
    ```js
    f.call(obj, a, b);   // this = obj
    f.apply(obj, [a, b]);
    const g = f.bind(obj); g(a, b);
    ```
    
    - `bind` permanently ties `this` for the returned function.
        
    - `new` over a bound function **overrides** the bound `this` (constructor semantics win), but keeps the preset arguments.
        
3. **Implicit binding**: `obj.method()`
    
    ```js
    obj.method(); // this = obj
    ```
    
    - Only the **last property reference** matters: in `a.b.c.fn()`, `this` is `c`.
        
    - If you **extract** the method (`const fn = obj.method; fn()`), you **lose** the binding.
        
4. **Default binding** (fallback)
    
    - In **strict mode**: `this = undefined`.
        
    - In **sloppy mode** (non-strict): `this = global object` (`window` in browsers, `global` in Node).
        

**Arrow functions**: ignore all four rules; `this` comes from the **enclosing** scope at creation time.

# Common call patterns and their `this`

- **Plain function call**: `fn()` → default binding.
    
- **Method call**: `obj.fn()` → implicit binding to `obj`.
    
- **Constructor**: `new Fn()` → brand-new instance.
    
- **`call/apply`**: `fn.call(obj, ...)` / `fn.apply(obj, args)` → explicit `obj`.
    
- **`bind`**: `const g = fn.bind(obj)` → `g()` always uses `obj` (unless used with `new`).
    

# Arrow functions and `this`

- Arrow functions **don’t have their own** `this`, `arguments`, `super`, or `new.target`.
    
- They capture `this` from the closest non-arrow function (or global/module scope if none).
    
- Passing an arrow to `call/apply/bind` **does not change** its `this`.
    

```js
const obj = {
  x: 10,
  normal() { return function() { return this?.x; }; },
  arrow()  { return () => this?.x; }
};
const n = obj.normal(); n();      // undefined (strict), because plain call lost binding
const a = obj.arrow();  a();      // 10, arrow captured obj’s this
```

# Why `this` often gets lost

- **Passing methods as callbacks**:
    
    ```js
    const hi = user.sayHi; // reference only
    hi(); // this = undefined (strict)
    ```
    
    Fix with `bind` or a wrapper:
    
    ```js
    setTimeout(user.sayHi.bind(user), 1000);
    setTimeout(() => user.sayHi(), 1000);
    ```
    
- **Destructuring methods**:
    
    ```js
    const { push } = Array.prototype;
    push.call(arr, 1); // must rebind explicitly; otherwise push(1) fails
    ```
    
- **Event handlers**:
    
    - `element.addEventListener('click', function(){ this === element })` → `true`
        
    - `element.addEventListener('click', () => { this /* from outer scope */ })` → not the element
        

# Classes and `this`

- Inside **instance methods**, `this` is the instance.
    
- Inside **static methods**, `this` is the class constructor itself (and in subclasses, `this` is the subclass—_polymorphic this_).
    
- **Class fields with arrow functions** capture instance `this` at construction time—useful for auto-binding event handlers:
    
    ```js
    class A {
      value = 1;
      handler = (e) => { console.log(this.value); }; // arrow captures instance
    }
    ```
    

# Getters/setters and prototypes (important nuance)

- In an accessor, `this` is the **receiver** (the object on which the property is accessed), not necessarily the object that **defines** the getter:
    
    ```js
    const proto = { get x() { return this.tag; } };
    const obj = Object.create(proto);
    obj.tag = 'OBJ';
    console.log(obj.x); // 'OBJ' — getter on proto sees `this = obj`
    ```
    
- `Reflect.get(target, 'x', receiver)` lets you control the `this` seen by a getter.
    

# Built-ins that accept a `thisArg`

Some array methods let you pass a `this` for the callback:

```js
arr.map(fn, thisArg);
arr.forEach(fn, thisArg);
```

Inside `fn`, `this === thisArg` (for non-arrow callbacks).

# Modules, scripts, Node, and top-level `this`

- In **ES modules** (browser & Node), top-level `this` is `undefined`.
    
- In **browser scripts** (non-module), top-level `this === window`.
    
- In **Node (CommonJS)** modules, top-level `this === module.exports`, not `global`.  
    (In the **Node REPL**, top-level `this === global`.)
    

# Timers and `this`

- `setTimeout(fn, ...)` calls `fn` as a plain function → default binding.
    
- If you pass a method reference (`setTimeout(obj.m, 0)`), you lose `this`. Use `bind` or a wrapper.
    

# `this` and DOM/event frameworks (quick notes)

- **React function components** don’t use `this`. Class components often used `this` + `.bind`.
    
- **Vue** component methods are bound to the component instance; don’t use arrow for methods that need reactivity context.
    
- **jQuery** event handlers (`function() {}`) set `this` to the element; arrows will capture outer `this` instead.
    

# Decision cheatsheet

- Need the function to see a specific object? → `call`/`apply`/`bind`.
    
- Passing a method as a callback? → wrap or `bind`.
    
- Need “outer `this`” inside a small inline function? → use an arrow.
    
- Writing utilities where `this` would confuse? → accept the object as a **normal parameter** instead of relying on `this`.
    

# Gotchas to remember

- `obj.method = obj.method.bind(obj)` creates a permanently bound function (can’t be “unbound”).
    
- `bind` also supports **partial application**: `fn.bind(ctx, arg1, arg2)` pre-sets leading args.
    
- Calling an **arrow function** with `call/apply` does **not** change `this`.
    
- `new (boundFn)` ignores the bound `this` but keeps bound arguments.
    
- In strict mode, `this` in plain functions is `undefined`—helps catch accidental globals.
    

# Mini examples

**Losing `this` by extraction**

```js
const user = { name: 'A', hi() { console.log(this.name); } };
const hi = user.hi;
hi();                // undefined (strict)
hi.call(user);       // 'A'
setTimeout(user.hi, 0);            // undefined
setTimeout(() => user.hi(), 0);    // 'A'
setTimeout(user.hi.bind(user), 0); // 'A'
```

**Arrow vs normal in events**

```js
btn.addEventListener('click', function () { console.log(this === btn); }); // true
btn.addEventListener('click', () => { console.log(this); }); // outer `this`, not btn
```

**Trailing throttle using closure `this/args`**

```js
function throttle(fn, ms) {
  let throttled = false, savedArgs, savedThis;
  return function (...args) {
    if (throttled) { savedArgs = args; savedThis = this; return; }
    throttled = true;
    fn.apply(this, args);
    setTimeout(() => {
      throttled = false;
      if (savedArgs) { fn.apply(savedThis, savedArgs); savedArgs = savedThis = null; }
    }, ms);
  };
}
```

(Notice how we **store** `this`/`args` because a later call would otherwise run with the **old** ones.)


----

## Other explanation

- Here is the way to think of it:
	1. It works like a function parameter
	2. Instead of being between the parentheses like other parameters, **it is the the thing to the left of the dot**
	3. Except when it is not

- It is #3 that really throws people, but that is best understood as short list of exceptions, not some fundamental property of `this`, and we'll get to it in a minute. Let's start with #1 and #2 which encompass 90% of the instances you will encounter `this`.

### A function parameter to the left of the dot

- So normally you put function parameters between the parentheses when you call it.
```js
function greet(user) {
  console.log('Hello,', user.name);
}

const sue = {
  name: 'Sue'
};

greet(sue);  // Hello, Sue
```

- We can easily rewrite this function to instead use `this`:
```js
function greetThis() {
  console.log('Hello,', this.name);
}

const bob = {
  name: 'Bob',
  greet: greetThis
};

bob.greet();  // Hello, Bob
```

- Simple enough, right? You can think of it as a way to identify the object that "owns" the function. That is the "context" part of "execution context". The "execution" part means "when the function is called or "executed". For example, we could define the function right inside the object:
```js
const ann = {
  name: 'Ann',
  greet() {
    console.log('Hello,', this.name);
  }
};

ann.greet();  // Hello, Ann
```

- But remember, `this` is a function parameter. It only has meaning _when the function is called_. If we move Ann's greet, `this` won't point to Ann anymore.
```js
const charles = {
  name: 'Charles',
  greet: ann.greet
};

charles.greet();  // Hello, Charles
```

- Okay. That is the basic behavior. If you understand that, you understand most of it. Let's talk about the weirdness and exceptions.

### When there is nothing to the left of the dot

- Okay, so what happens when there is nothing to the left of the dot? Well, the unfortunate answer is _it depends_.
</br>
- If you are in "strict mode" (which includes ES modules), then `this` will very sensibly be `undefined`.
```js
'strict mode';

greetThis();  // TypeError: Cannot read properties of undefined (reading 'name')
```

- This works exactly like regular function parameters when you don't pass them in between the parentheses. Unfortunately, if you are _not_ in strict mode `this` very weirdly defaults to the "global context", i.e. `window` in the browser and `global` in Node. We could talk more about this, but all you really need to understand is that it is silly and weird and if it happens, you screwed something up.

- This most often comes up when passing an object method as a callback.
```js
'strict mode';

setTimeout(ann.greet, 1000);

// TypeError: Cannot read properties of undefined (reading 'name')
```
- Here we passed our function to `setTimeout`, but we did not pass Ann. So when the greet method gets called by `setTimeout`, there will not be anything to the left of the dot. This is often solved by wrapping everything in another function.
```js
setTimeout(function() {
  ann.greet();
}, 1000);

// Hello, Ann
```
- Now when greet is called, Ann will be to the left of the dot.

### In a class constructor

```js
class User {
  constructor(name) {
    this.name = name;
  }
}

const theresa = new User('Theresa');
```

- Okay. So what gives here? We used `this`, but there was no dot when we called User.
- So the `new` keyword causes functions to be called in "constructor mode". You can think of this as secretly re-assigning `this` to a new object and then returning it at the end.

```js
class User {
  constructor(name) {
    // this = {}
    this.name = name;
    // return this
  }
}
```

### Call, apply, and bind

- Functions in JavaScript have three methods which let you mess with how they are called. They all take a value for `this` as their first argument, which means they are often associated with `this`, but people often forget that the later arguments are all values for the _other_ function parameters.

```js
function greetCustom(greeting) {
  console.log(greeting, this.name);
}

// Normal parameters
ann.greet = greetCustom;
ann.greet('Hey');  // Hey Ann

// Call
greetCustom.call(bob, 'Ciao');  // Ciao Bob

// Apply
greetCustom.apply(sue, ['Yo']);  // Yo Sue

// Bind
const greetCharles = greetCustom.bind(charles, 'Salutations');
greetCharles();  // Salutations Charles
```

- You can look up these three methods if you want, but for the most part you won't need them in modern JavaScript. Nonetheless, it is worth mentioning that if you use them, they change the normal rules for function parameters (including `this`).

### Arrow functions

- Finally, it is worth mentioning that arrows functions do not have a `this`. If you use `this` inside an arrow function, it will not take any value related to how you called the arrow function. Instead it will behave like any variable which is not defined in a nested scope. Variable lookup will fallback to the wrapping scope and use the one defined there.

```js
'use strict';

function greetNested() {
   // this is defined in this function scope
   // but has no value when called
   function greet() {
     console.log('Hello,', this.name);
   }

   greet();
}

ann.greet = greetNested;
ann.greet();  // TypeError: Cannot read properties of undefined (reading 'name')

function greetNestedArrow() {
   // this does not exist in this arrow scope
   // and so falls back to the wrapping function
   const greet = () => console.log('Hello,', this.name);
   greet();
}

ann.greet = greetNestedArrow;
ann.greet();  // Hello, Ann
```

- And that's it. Just remember:

> `this` is the thing to the left of the dot when a function is called... except when its not