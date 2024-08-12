It is also possible to apply the static keyword directly on the class level. When a class has been defined as static, it is not creatable using the new keyword, and it can contain only members or data fields marked with the static keyword. If this is not the case, you receive compiler errors.  
```ad-note
Note recall that a class (or structure) that exposes only static functionality is often termed a utility class. When designing a utility class, it is good practice to apply the static keyword to the class definition.
```