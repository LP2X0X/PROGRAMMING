---
tags: cpp, term, fundamental
---
- An [[C++/1 - Basic C++/Terms/Operators/Operator|operator]] (or function) that has some observable effect **beyond producing a return value** is said to have a **side effect**.
	- For example, when `x = 5` is evaluated, the assignment operator has the side effect of assigning the value `5` to variable `x`. The changed value of `x` is observable (e.g. by printing the value of `x`) even after the operator has finished executing.
	- `std::cout << 5` has the side effect of printing `5` to the console. We can observe the fact that `5` has been printed to the console even after `std::cout << 5` has finished executing.

---

- Both `operator=` and `operator<<` (when used to output values to the console) **return their left operand**. Thus, `x = 5` returns `x`, and `std::cout << 5` returns `std::cout`. This is done so that these operators can be chained.
	- For example, `x = y = 5` evaluates as `x = (y = 5)`. First `y = 5` assigns `5` to `y`. This operation then returns `y`, which can then be assigned to `x`.
	- `std::cout << "Hello " << "world!"` evaluates as `(std::cout << "Hello ") << "world!"`. This first prints `"Hello "` to the console. This operation returns `std::cout`, which can then be used to print `"world!"` to the console as well