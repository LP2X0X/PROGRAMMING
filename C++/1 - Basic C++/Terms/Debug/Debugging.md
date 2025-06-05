---
tags: cpp, tip
---

Debugging involves identifying, analyzing, and fixing bugs or errors in software. Here are some common debugging terms you might encounter:

### 1. **Breakpoint**
   - A deliberate stopping or pausing place in a program set by a developer or a debugger. When the program execution reaches a breakpoint, it pauses, allowing you to inspect the state of the application at that point.

### 2. **Watchpoint (or Data Breakpoint)**
   - Similar to a breakpoint, but instead of pausing the program at a specific line of code, it pauses execution when the value of a particular variable changes.

### 3. **Stack Trace**
   - A report of the active stack frames at a certain point in time during program execution, typically when an exception is thrown. It shows the call stack and helps identify where the error occurred.

### 4. **Step Over**
   - A debugger command that allows you to execute the next line of code. If the line is a function call, the debugger executes the entire function and stops at the next line of code in the current function.

### 5. **Step Into**
   - A debugger command that allows you to go inside a function. If the next line of code is a function call, the debugger will enter the function and stop at its first line.

### 6. **Step Out**
   - A debugger command that continues executing the remaining lines of the current function and pauses execution when it returns to the caller function.

### 7. **Watch Expression**
   - A feature in debugging where you monitor a specific variable or expression. The debugger will display the value of the watched variable or expression as you step through the code.

### 8. **Exception**
   - An error or unexpected event that occurs during the execution of a program. Exceptions can be caught and handled to prevent the program from crashing.

### 9. **Core Dump (or Crash Dump)**
   - A file that records the memory of a running program at a specific time, usually when the program crashes. It can be analyzed to determine the cause of the crash.

### 10. **Segmentation Fault (Segfault)**
   - A specific kind of error caused by accessing memory that “does not belong to you.” It usually results in a program crash and can be difficult to debug without proper tools.

### 11. **Null Pointer Dereference**
   - An error that occurs when a program attempts to use a pointer that has not been initialized or has been set to `NULL`. Dereferencing a null pointer typically results in a segmentation fault.

### 12. **Memory Leak**
   - A situation where a program allocates memory but does not free it, causing the program to consume more and more memory over time, which can eventually lead to system slowdowns or crashes.

### 13. **Heisenbug**
   - A type of bug that seems to disappear or change its behavior when one tries to study it, such as by using debugging tools.

### 14. **Race Condition**
   - An error that occurs when the outcome of a program depends on the sequence or timing of uncontrollable events, such as the order in which threads execute.

### 15. **Deadlock**
   - A situation in multithreading where two or more threads are blocked forever, each waiting on the other to release a resource.

### 16. **Assertion**
   - A statement in code that checks if a condition is true. If the condition is false, the program will typically terminate with an error message. Assertions are used to catch programming errors during development.

### 17. **Hot Code**
   - Code that is executed frequently and may be a candidate for optimization or special attention during debugging due to its impact on performance.

### 18. **Cold Code**
   - Code that is rarely executed, possibly representing edge cases or error-handling scenarios. Bugs in cold code can be hard to find because the code paths are infrequently tested.

### 19. **Backtrace**
   - Similar to a stack trace, it is a list of the function calls that led to a particular point in the execution of a program, usually provided after a crash or exception.

### 20. **Log File**
   - A file where a program writes output for debugging or auditing purposes. Logs are used to track the execution flow and identify where errors occur.

### 21. **GDB**
   - Short for GNU Debugger, it's a powerful tool used for debugging programs, especially in Unix-like operating systems. It allows developers to inspect the runtime behavior of a program, set breakpoints, and more.

### 22. **Valgrind**
   - A programming tool for memory debugging, memory leak detection, and profiling. It is particularly useful for detecting memory management bugs.

These terms are integral to the process of debugging and are commonly used by developers when diagnosing and fixing issues in software.