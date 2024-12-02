- In general programming, an **aggregate data type** (also called an **aggregate**) is any type that can contain **multiple data members**. Some types of aggregates allow members to have different types (e.g. structs), while others require that all members must be of a single type (e.g. arrays).

---

### Formal definition
- An aggregate is an array or a class (clause 9) with no user-declared constructors (12.1), no private or protected non-static data members (clause 11), no base classes (clause 10), and no virtual functions (10.3).
	- This does not mean an aggregate class cannot have constructors, in fact it can have a default constructor and/or a copy constructor as long as they are implicitly declared by the compiler, and not explicitly by the user
	- No private or protected _**non-static data members**_. You can have as many private and protected member functions (but not constructors) as well as as many private or protected _**static**_ data members and member functions as you like and not violate the rules for aggregate classes
	- An aggregate class can have a user-declared/user-defined copy-assignment operator and/or destructor
	- An array is an aggregate even if it is an array of non-aggregate class type.