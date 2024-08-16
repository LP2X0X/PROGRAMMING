- The return value form the main function is sometimes called a **status code** or **exit code** or rarely **return code**.
- When the return statement is executed, the function will return immediately (even for non-value return function).
- By convention, a status code of 0 indicate that the program ran normally.

---
### For advanced readers
- The C++ standard only defines the meaning of 3 status codes: 0, EXIT_SUCCESS, and EXIT_FAILURE. 0 and EXIT_SUCCESS both mean the program executed successfully. EXIT_FAILURE means the program did not execute successfully.
- EXIT_SUCCESS and EXIT_FAILURE are preprocessor macros defined in the \<cstdlib\> header:

```cpp
#include <cstdlib> // for EXIT_SUCCESS and EXIT_FAILURE

int main()
{
    return EXIT_SUCCESS;
}
```
- If you want to maximize portability, you should only use 0 or EXIT_SUCCESS to indicate a successful termination, or EXIT_FAILURE to indicate an unsuccessful termination.
- The status code is passed back to the operating system. The OS will typically make the status code available to whichever program launched the program returning the status code. This provides a crude mechanism for any program launching another program to determine whether the launched program ran successfully or not.
---