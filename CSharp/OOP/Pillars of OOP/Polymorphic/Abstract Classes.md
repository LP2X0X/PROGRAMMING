- At first glance, it might seem strange to define a class that you cannot directly create an instance of. Recall, however, that base classes (abstract or not) are useful, in that they contain all the common data and functionality of derived types. Using this form of abstraction, you are able to model that the “idea” of an employee is completely valid; it is just not a concrete entity. Also understand that although you cannot directly create an instance of an abstract class, it is still assembled in memory when derived classes are created. Thus, it is perfectly fine (and common) for abstract classes to define any number of constructors that are called indirectly when derived classes are allocated.
- When a class has been defined as an abstract base class (via the abstract keyword), it may define any number of abstract members. Abstract members can be used whenever you want to define a member that does not supply a default implementation but must be accounted for by each derived class. By doing so, you enforce a polymorphic interface on each descendant, leaving them to contend with the task of providing the details behind your abstract methods.
```ad-note
Abstract members do not provide any implementation whatsoever.
Abstract methods can be defined only in abstract classes. If you attempt to do otherwise, you will be issued a compiler error.
```

---

- Although it is not possible to directly create an instance of an abstract base class (the Shape), you are able to freely store references to any subclass with an abstract base variable. Therefore, when you are creating an array of Shapes, the array can hold any object deriving from the Shape base class (if you attempt to place Shape-incompatible objects into the array, you receive a compiler error).