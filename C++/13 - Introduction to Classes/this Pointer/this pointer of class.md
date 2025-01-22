Inside every member function, the keyword **this** is a const pointer that holds the address of the current implicit object.

---

### How is `this` set?

Let’s take a closer look at this function call:

```cpp
simple.setID(2);
```

Although the call to function `setID(2)` looks like it only has one argument, it actually has two! When compiled, the compiler rewrites the expression `simple.setID(2);` as follows:

```cpp
Simple::setID(&simple, 2); // note that simple has been changed from an object prefix to a function argument!
```

Note that this is now just a standard function call, and the object `simple` (which was formerly an object prefix) is now passed by address as an argument to the function.

But that’s only half of the answer. Since the function call now has an added argument, the member function definition also needs to be modified to accept (and use) this argument as a parameter. Here’s our original member function definition for `setID()`:

```cpp
void setID(int id) { m_id = id; }
```

How the compiler rewrites functions is an implementation-specific detail, but the end-result is something like this:

```cpp
static void setID(Simple* const this, int id) { this->m_id = id; }
```

Note that our `setId` function has a new leftmost parameter named `this`, which is a const pointer (meaning it cannot be re-pointed, but the contents of the pointer can be modified). The `m_id` member has also been rewritten as `this->m_id`, utilizing the `this` pointer.

---

### Returning *this

It can sometimes be useful to have a member function return the implicit object as a return value. The primary reason to do this is to allow member functions to be “chained” together, so several member functions can be called on the same object in a single expression! This is called **function chaining** (or **method chaining**).

Consider this common example where you’re outputting several bits of text using `std::cout`:

```cpp
std::cout << "Hello, " << userName;
```

The compiler evaluates the above snippet like this:

```cpp
(std::cout << "Hello, ") << userName;
```

First, `operator<<` uses `std::cout` and the string literal `"Hello, "` to print `"Hello, "` to the console. However, since this is part of an expression, `operator<<` also needs to return a value (or `void`). If `operator<<` returned `void`, you’d end up with this as the partially evaluated expression:

```cpp
void{} << userName;
```

which clearly doesn’t make any sense (and the compiler would throw an error). Instead, `operator<<` returns the stream object that was passed in, which in this case is `std::cout`. That way, after the first `operator<<` has been evaluated, we get:

```cpp
(std::cout) << userName;
```

which then prints the user’s name.

This way, we only need to specify `std::cout` once, and then we can chain as many pieces of text together using `operator<<` as we want. Each call to `operator<<` returns `std::cout` so the next call to `operator<<` uses `std::cout` as the left operand.