- When a base class defines protected data or protected members, it establishes a set of items that can be accessed directly by any descendant.

```ad-note 
Convention is that protected members are named PascalCased (EmpName) and not underscorecamelCase (_empName). This is not a requirement of the language, but a common code style.
```

```ad-warning
The benefit of defining protected members in a base class is that derived types no longer have to access the data indirectly using public methods or properties. The possible downfall, of course, is that when a derived type has direct access to its parent’s internal data, it is possible to accidentally bypass existing business rules found within public properties. When you define protected members, you are creating a level of trust between the parent class and the child class, as the compiler will not catch any violation of your type’s business rules.
```