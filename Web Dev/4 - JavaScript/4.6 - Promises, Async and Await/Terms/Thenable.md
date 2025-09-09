---
tags: 
 - js
 - object
 - promise
 - advance
---

## 1. What is a Thenable?

A **thenable** is any object that has a `.then` method that follows the same convention as a `Promise`.

```js
let thenable = {
  then(resolve, reject) {
    resolve("I look like a promise!");
  }
};
```

So:

- A real `Promise` is a thenable.
    
- But not all thenables are Promises — they just need a `.then(resolve, reject)` method.
    

The idea is that 3rd-party libraries may implement “promise-compatible” objects of their own. They can have an extended set of methods, but also be compatible with native promises, because they implement `.then`.

---

## 2. Why Thenables Exist

The **Promise/A+ spec** says that `Promise.resolve(x)` will:

- If `x` is a promise → return `x`.
    
- If `x` is a thenable → treat it like a promise and “assimilate” it.
    
- Otherwise → wrap it in a resolved promise.
    

---

## 3. Example: Using a Thenable

```js
let thenable = {
  then(resolve, reject) {
    setTimeout(() => resolve("done!"), 1000);
  }
};

Promise.resolve(thenable).then(value => console.log(value));
// after 1s → "done!"
```

Even though `thenable` is not an actual `Promise`, `Promise.resolve` treats it as one because it has a `.then`.

---

## 4. Why Useful?

- Lets libraries create **“promise-like” objects** without depending on native `Promise`.
    
- Enables **interoperability** between different async libraries.
    

Example: older async libs like jQuery’s `$.Deferred` can be wrapped and used like promises.

---

## 5. In Async/Await

`await` also works with thenables:

```js
let thenable = {
  then(resolve) {
    setTimeout(() => resolve("await works too!"), 500);
  }
};

(async () => {
  let result = await thenable;
  console.log(result); // "await works too!"
})();
```

---

✅ **Summary**

- A **thenable** is any object with a `.then(resolve, reject)` method.
    
- Promises are thenables, but thenables don’t have to be promises.
    
- JS promise machinery (`Promise.resolve`, `await`) automatically works with them.
    