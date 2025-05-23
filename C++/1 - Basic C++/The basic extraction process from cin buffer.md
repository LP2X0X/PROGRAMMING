---
tags: cpp, fundamental
---

#### The basic extraction process
Here’s a simplified view of how operator `>>` works for input:

1. First, leading whitespace (spaces, tabs, and newlines at the front of the buffer) is discarded from the input buffer. This will discard any unextracted newline character remaining from a prior line of input.
2. If the input buffer is now empty, operator `>>` will wait for the user to enter more data. Leading whitespace is again discarded.
3. Operator `>>` then extracts as many consecutive characters as it can, until it encounters either a newline character (representing the end of the line of input) or **a character that is not valid for the variable being extracted to (this means that the extracted characters will depend on the variable's type which will contain the value).**

````ad-example
```cpp
#include <iostream>  // for std::cout and std::cin

int main()
{
    std::cout << "Enter a number: "; // ask user for a number
    int x{}; // define variable x to hold user input
    std::cin >> x; // get number from keyboard and store it in variable x 
    std::cout << "You entered " << x << '\n';

    return 0;
}
```
If you input the number `3.2`, the `3` gets extracted, but `.` is an invalid character, so extraction stops here. The `.2` remains for a future extraction attempt.
If you change the type of x from int to float, input the number `3.2`, then all will be extracted.
````

The result of the extraction is as follows:

- If any characters were extracted in step 3 above, extraction is a success. The extracted characters are converted into a value that is then assigned to the variable.
- If no characters could be extracted in step 3 above, extraction has failed. The object being extracted to is assigned the value `0` (as of C++11), and any future extractions will immediately fail (until `std::cin` is cleared).

Any non-extracted characters (including newlines) remain available for the next extraction attempt.