---
tags:
  - js
  - term
  - advanced
---

### Step 1. Variables

- In JavaScript, every running function, code block `{...}`, and the script as a whole have an internal (hidden) associated object known as the _Lexical Environment_.
- The Lexical Environment object consists of two parts:
	1. _Environment Record_ – an object that stores all local variables as its properties (and some other information like the value of `this`).
	2. A reference to the _outer lexical environment_, the one associated with the outer code.

```ad-note
**A “variable” is just a property of the special internal object, `Environment Record`. “To get or change a variable” means “to get or change a property of that object”.**
```

</br>

- In this simple code without functions, there is only one Lexical Environment:
![[Pasted image 20250801140024.png|500]]

- This is the so-called _global_ Lexical Environment, associated with the whole script.
- On the picture above, the rectangle means Environment Record (variable store) and the arrow means the outer reference. The global Lexical Environment has no outer reference, that’s why the arrow points to `null`.

</br>

- As the code starts executing and goes on, the Lexical Environment changes:
![[Pasted image 20250801140312.png|500]]

- Rectangles on the right-hand side demonstrate how the global Lexical Environment changes during the execution:
	1. When the script starts, the Lexical Environment is pre-populated with all declared variables.
	    - Initially, they are in the “Uninitialized” state. That’s a special internal state, it means that the engine knows about the variable, but it cannot be referenced until it has been declared with `let`. It’s almost the same as if the variable didn’t exist.
	2. Then `let phrase` definition appears. There’s no assignment yet, so its value is `undefined`. We can use the variable from this point forward.
	3. `phrase` is assigned a value.
	4. `phrase` changes the value.


```ad-summary
A variable is a property of a special internal object, associated with the currently executing block/function/script.
Working with variables is actually working with the properties of that object.
```

----

### Step 2. Function Declarations

- A function is also a value, like a variable. **The difference is that a Function Declaration is instantly fully initialized.**
- When a Lexical Environment is created, a Function Declaration immediately becomes a ready-to-use function (unlike `let`, that is unusable till the declaration).
- That’s why we can use a function, declared as Function Declaration, even before the declaration itself.

</br>

- For example, here’s the initial state of the global Lexical Environment when we add a function:
![[Pasted image 20250801141101.png|600]]
- Naturally, this behavior only applies to Function Declarations, not Function Expressions where we assign a function to a variable, such as `let say = function(name)...`.

----

### Step 3. Inner and outer Lexical Environment

- When **a function runs**, at the beginning of the call, **a new Lexical Environment is created automatically to store local variables and parameters of the call**.

</br>

- For instance, for `say("John")`, it looks like this (the execution is at the line, labelled with an arrow):
![[Pasted image 20250801141313.png|700]]
- During the function call we have two Lexical Environments: the inner one (for the function call) and the outer one (global):
	- The inner Lexical Environment corresponds to the current execution of `say`. It has a single property: `name`, the function argument. We called `say("John")`, so the value of the `name` is `"John"`.
	- The outer Lexical Environment is the global Lexical Environment. It has the `phrase` variable and the function itself.
- The inner Lexical Environment has a reference to the `outer` one.
- In this example the search proceeds as follows:
	- For the `name` variable, the `alert` inside `say` finds it immediately in the inner Lexical Environment.
	- When it wants to access `phrase`, then there is no `phrase` locally, so it follows the reference to the outer Lexical Environment and finds it there.
![[Pasted image 20250801141647.png|700]]

```ad-important
**When the code wants to access a variable – the inner Lexical Environment is searched first, then the outer one, then the more outer one and so on until the global one.**
```

```ad-note
If a variable is not found anywhere, that’s an error in strict mode (without `use strict`, an assignment to a non-existing variable creates a new global variable, for compatibility with old code).
```

----

### Step 4. Returning a function
- Let's look at this example:
	```js
	function makeCounter() {
	  let count = 0;
	
	  return function() {
	    return count++;
	  };
	}
	
	let counter = makeCounter();
	```

</br>

- At the beginning of each `makeCounter()` call, a new Lexical Environment object is created, to store variables for this `makeCounter` run.
- So we have two nested Lexical Environments, just like in the example above:
![[Pasted image 20250801142009.png|700]]

</br>

- What’s different is that, during the execution of `makeCounter()`, a tiny nested function is created of only one line: `return count++`. We don’t run it yet, **only create**.
- All functions remember the Lexical Environment in which they were made. Technically, there’s no magic here: all functions have the hidden property named `[[Environment]]`, that keeps the reference to the Lexical Environment where the function was created:
![[Pasted image 20250801142352.png|700]]
- So, `counter.[[Environment]]` has the reference to `{count: 0}` Lexical Environment. That’s how the function remembers where it was created, no matter where it’s called. The `[[Environment]]` reference is set once and forever at function creation time.

</br>

- Later, when `counter()` is called, a new Lexical Environment is created for the call, and its outer Lexical Environment reference is taken from `counter.[[Environment]]`:
![[Pasted image 20250801142859.png|700]]

- Now when the code inside `counter()` looks for `count` variable, it first searches its own Lexical Environment (empty, as there are no local variables there), then the Lexical Environment of the outer `makeCounter()` call, where it finds and changes it.
![[Pasted image 20250801143548.png|700]]

```ad-important
**A variable is updated in the Lexical Environment where it lives.**
```

----

### Garbage collection

- Usually, a Lexical Environment is removed from memory with all the variables after the function call finishes. That’s because there are no references to it. As any JavaScript object, it’s only kept in memory while it’s reachable.
- **However, if there’s a nested function that is still reachable after the end of a function, then it has `[[Environment]]` property that references the lexical environment.**
- In that case the Lexical Environment is still reachable even after the completion of the function, so it stays alive.

</br>

- For example:
	```js
	function f() {
	  let value = 123;
	
	  return function() {
	    alert(value);
	  }
	}
	
	let g = f(); // g.[[Environment]] stores a reference to the Lexical Environment
	// of the corresponding f() call
	```

- Please note that if `f()` is called many times, and resulting functions are saved, then all corresponding Lexical Environment objects will also be retained in memory. In the code below, all 3 of them:
	```js
	function f() {
	  let value = Math.random();
	
	  return function() { alert(value); };
	}
	
	// 3 functions in array, every one of them links to Lexical Environment
	// from the corresponding f() run
	let arr = [f(), f(), f()];
	```

- A Lexical Environment object dies when it becomes unreachable (just like any other object). In other words, it exists only while there’s at least one nested function referencing it.
- In the code below, after the nested function is removed, its enclosing Lexical Environment (and hence the `value`) is cleaned from memory:
	```js
	function f() {
	  let value = 123;
	
	  return function() {
	    alert(value);
	  }
	}
	
	let g = f(); // while g function exists, the value stays in memory
	
	g = null; // ...and now the memory is cleaned up
	```

----

### Lexical Environment with loop

- A new **inner lexical environment is created per iteration**, but **only**:
    - In `for`, `for...in`, `for...of`
    - When using `let` or `const`
- All other cases share the same environment across iterations.


|Loop Type|With `var`|With `let` / `const`|Lexical Environment per Iteration?|
|---|---|---|---|
|`for`|❌ Shared (1 env)|✅ New per iteration|✅ Yes, with `let`/`const`|
|`while`|❌ Shared|❌ Shared|❌ No|
|`do...while`|❌ Shared|❌ Shared|❌ No|
|`for...in`|❌ Shared|✅ New per iteration|✅ Yes, with `let`/`const`|
|`for...of`|❌ Shared|✅ New per iteration|✅ Yes, with `let`/`const`|


````ad-tip
You can override the limitation with shared outer Lexical Environment by create a local variable for it in some cases like this.
```javascript
function makeArmy() {
  let shooters = [];

  let i = 0;
  while (i < 10) {
      let j = i;
      let shooter = function() { // shooter function
        alert( j ); // should show its number
      };
    shooters.push(shooter);
    i++;
  }

  return shooters;
}

let army = makeArmy();

// Now the code works correctly
army[0](); // 0
army[5](); // 5
```
````