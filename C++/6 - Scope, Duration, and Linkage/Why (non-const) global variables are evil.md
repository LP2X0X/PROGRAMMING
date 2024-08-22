Non-const global variables are often considered "evil" in software development for several reasons related to maintainability, debugging difficulty, and potential for introducing errors. While they can be useful in certain situations, their drawbacks typically outweigh the benefits, especially in larger, more complex projects. Here’s why:

### 1. **Uncontrolled Access**
   - **Global Scope**: Non-const global variables can be accessed and modified by any part of the program, which makes it difficult to track where and how they are changed. This lack of control can lead to unexpected side effects.
   - **Lack of Encapsulation**: Global variables break the principle of encapsulation, where the internal state of an object or function should be hidden from other parts of the program. Encapsulation is crucial for creating modular and maintainable code.

### 2. **Difficult to Debug**
   - **Side Effects**: Since any part of the program can modify a global variable, bugs can be hard to trace. A change in a global variable might affect many different parts of the program, making it challenging to identify the source of an issue.
   - **State Management**: Managing the state of global variables across multiple functions or modules can be tricky, especially if the program’s logic becomes complex. This can result in "spaghetti code," where the program flow is tangled and difficult to follow.

### 3. **Reduced Reusability**
   - **Tight Coupling**: Global variables often lead to tightly coupled code, where different parts of the program are overly dependent on each other. This reduces the reusability of code, as functions that rely on global variables are difficult to isolate and reuse in different contexts.
   - **Module Interdependence**: When modules depend on global variables, it becomes harder to reuse or test individual modules independently. This tight coupling makes refactoring and maintaining the codebase more difficult.

### 4. **Concurrency Issues**
   - **Race Conditions**: In multi-threaded programs, global variables can lead to race conditions, where multiple threads attempt to read or modify the variable simultaneously. This can result in inconsistent or unpredictable behavior unless proper synchronization (e.g., mutexes) is used.
   - **Synchronization Overhead**: To avoid race conditions, you often need to introduce synchronization mechanisms, which add complexity and can reduce performance.

### 5. **Namespace Pollution**
   - **Name Conflicts**: Global variables occupy the global namespace, increasing the risk of name conflicts, especially in large projects or when integrating third-party libraries. This can lead to subtle bugs that are difficult to diagnose.
   - **Maintainability Issues**: As a project grows, the likelihood of name clashes increases, leading to a need for careful naming conventions or other workarounds, which can make the code harder to maintain.

### 6. **Testing Challenges**
   - **Harder to Test**: Functions that rely on global variables are harder to test in isolation (unit testing). They introduce hidden dependencies, which make it difficult to write predictable and repeatable tests.
   - **Mocking Issues**: In testing, you often need to mock dependencies. With global variables, this becomes more difficult since their state persists across different tests unless manually reset, increasing the complexity of test setup and teardown.

### 7. **Security and Safety Concerns**
   - **Security Risks**: If global variables are used to store sensitive data, they can pose a security risk since they are accessible from anywhere in the program. This increases the surface area for potential vulnerabilities.
   - **Undefined Behavior**: Improper use of global variables can lead to undefined behavior, especially if they are accessed before being initialized or modified in unexpected ways.

### **Alternatives to Global Variables**

Instead of using non-const global variables, consider the following alternatives:

1. **Pass Parameters**: Pass necessary data to functions through parameters instead of relying on global variables.
2. **Class Members**: Encapsulate state within classes and use instance variables instead of globals. This confines state changes to the class’s methods.
3. **Singleton Pattern**: If you need a single shared instance of an object, consider using the Singleton pattern, though be aware of its own potential downsides.
4. **Dependency Injection**: Use dependency injection to pass shared objects or services to the parts of the program that need them.
5. **Local Static Variables**: For data that needs to persist across function calls but doesn't need to be global, consider using local static variables.

### **Summary**

Non-const global variables are considered "evil" because they break encapsulation, make code harder to debug, reduce reusability, introduce concurrency risks, pollute the global namespace, complicate testing, and can lead to security vulnerabilities. While they can be useful in some cases, their drawbacks usually outweigh their benefits, especially in larger, more complex projects. It’s generally better to use more controlled and modular approaches to manage state and share data across different parts of a program.