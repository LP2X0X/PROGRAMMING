- C++ allows us to define our own namespaces via the `namespace` keyword. Namespaces that you create in your own programs are casually called **user-defined namespaces** (though it would be more accurate to call them **program-defined namespaces**).

```ad-tip
- Some reasons to prefer namespace names starting with a capital letter:
	- It is convention to name program-defined types starting with a capital letter. Using the same convention for program-defined namespaces is consistent (especially when using a qualified name such as `Foo::x`, where `Foo` could be a namespace or a class type).
	- It helps prevent naming collisions with other system-provided or library-provided lower-cased names.
	- The C++20 standards document uses this style.
	- The C++ Core guidelines document uses this style.
```