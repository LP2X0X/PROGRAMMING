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
