-  An **assertion** is an expression that will be true unless there is a bug in the program. If the expression evaluates to `true`, the assertion statement does nothing. If the conditional expression evaluates to `false`, an error message is displayed and the program is terminated (via `std::abort`). This error message typically contains the expression that failed as text, along with the name of the code file and the line number of the assertion. This makes it very easy to tell not only what the problem was, but where in the code the problem occurred. This can help with debugging efforts immensely.

--- 
### Runtime assertions
- In C++, runtime assertions are implemented via the **assert** preprocessor macro, which lives in the \<cassert\> header.
```cpp
#include <cassert> // for assert()
#include <cmath> // for std::sqrt
#include <iostream>

double calculateTimeUntilObjectHitsGround(double initialHeight, double gravity)
{
  assert(gravity > 0.0); // The object won't reach the ground unless there is positive gravity.

  if (initialHeight <= 0.0)
  {
    // The object is already on the ground. Or buried.
    return 0.0;
  }

  return std::sqrt((2.0 * initialHeight) / gravity);
}

int main()
{
  std::cout << "Took " << calculateTimeUntilObjectHitsGround(100.0, -9.8) << " second(s)\n";

  return 0;
}
```

````ad-tip
There’s a little trick you can use to make your assert statements more descriptive. Simply add a string literal joined by a logical AND:
```cpp
assert(found && "Car could not be found in database");
```
Result:
`Assertion failed: found && "Car could not be found in database", file C:\\VCProjects\\Test.cpp, line 34`
````

````ad-tip
Assertions are also sometimes used to document cases that were not implemented because they were not needed at the time the programmer wrote the code:
```cpp
assert(moved && "Need to handle case where student was just moved to another classroom");
```
That way, if a developer encounters a situation where this case is needed, the code will fail with a useful error message, and the programmer can then determine how to implement that case.
````

---

### static _assert
- C++ also has another type of assert called `static_assert`. A **static_assert** is an assertion that is checked at compile-time rather than at runtime, with a failing `static_assert` causing a compile error. Unlike assert, which is declared in the \<cassert\> header, static_assert is a keyword, so no header needs to be included to use it.

- A few useful notes about `static_assert`:
	- Because `static_assert` is evaluated by the compiler, the condition must be a constant expression.
	- `static_assert` can be placed anywhere in the code file (even in the global namespace).
	- `static_assert` is not deactivated in release builds (like normal `assert` is).
	- Because the compiler does the evaluation, there is no runtime cost to a `static_assert`.