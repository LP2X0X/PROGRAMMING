- Prefer to put your class definitions in a header file with the same name as the class. Trivial member functions (such as access functions, constructors with empty bodies, etc…) can be defined inside the class definition.
- Prefer to define non-trivial member functions in a source file with the same name as the class.

---

- Member functions defined _inside_ the class definition are implicitly inline. Inline functions are exempt from the one definition per program part of the one-definition rule.
- Member functions defined _outside_ the class definition are not implicitly inline (and thus are subject to the one definition per program part of the one-definition rule). This is why such functions are usually defined in a code file (where they will only have one definition across the whole program).

---

### Inline expansion of member functions
- The compiler must be able to see a full definition of a function in order to perform inline expansion. Most often, such functions (e.g. access functions) are defined inside the class definition. However, if you want to define a member function outside the class definition, but still want it to be eligible for inline expansion, you can define it as an inline function just below the class definition (in the same header file). That way the definition of the function is accessible to anybody who \#includes the header.