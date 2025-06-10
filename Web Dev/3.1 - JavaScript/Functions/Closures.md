- A closure is **the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment)**. In other words, a closure gives a function access to its outer scope. In JavaScript, closures are created every time a function is created, at function creation time.
- The key ingredients for a _useful_ closure are the following:
	- A parent scope that defines some variables or functions. It should have a clear lifetime, which means it should finish execution at some point. Any scope that's not the global scope satisfies this requirement; this includes blocks, functions, modules, and more.
	- An inner scope defined within the parent scope, which refers to some variables or functions defined in the parent scope.
	- The inner scope manages to survive beyond the lifetime of the parent scope. For example, it is saved to a variable that's defined outside the parent scope, or it's returned from the parent scope (if the parent scope is a function).
	- Then, when you call the function outside of the parent scope, you can still access the variables or functions that were defined in the parent scope, even though the parent scope has finished execution.

```js
// The outer function defines a variable called "name"
const pet = function (name) {
  const getName = function () {
    // The inner function has access to the "name" variable of the outer function
    return name;
  };
  return getName; // Return the inner function, thereby exposing it to outer scopes
};
const myPet = pet("Vivie");

console.log(myPet()); // "Vivie"

```