The **input/output library** (io library) is part of the C++ standard library that deals with basic input and output. The _io_ part of _iostream_ stands for _input/output_.

---

### std::cout
- The cout ***variable*** allows us to send the data to the console to be printed as text. We use _std::cout_, along with the **insertion operator (`<<`)**, to send the text _Hello world!_ to the console to be printed.
- Statements in our program request the output be sent to the console. However, that output is typically not sent to the console immediately. Instead, the requested output “gets in line”, and is stored in a region of memory set aside to collect such requests (called a **buffer**). 
- Periodically, the buffer is **flushed**, meaning all of the data collected in the buffer is transferred to its destination (in this case, the console).

---

### std::endl
- If we want to print separate lines of output to the console, we need to tell the console to move the cursor to the next line. We can do that by outputting a newline. A **newline** is an OS-specific character or sequence of characters that moves the cursor to the start of the next line.
- One way to output a newline is to output `std::endl` (which stands for “end line”).

```ad-tip
Best practice: Prefer \n over str::endl because **std::cout << std::endl** inserts a new line **and flushes the stream(output buffer)**, whereas **std::cout << “\n”** just inserts a new line.
```

---

### std::cin
- This ***variable*** read input from the keyboard. We typically use the extraction operator `>>` to put the input data in a variable (which can then be used in subsequent statements).
- Inputting data is also a two stage process:
	- The individual characters you enter as input are added to the end of an input buffer (inside `std::cin`). The enter key (pressed to submit the data) is also stored as a `'\n'` character.
	- The extraction operator ‘>>’ removes characters from the front of the input buffer and converts them into a value that is assigned to the associated variable. This variable can then be used in subsequent statements.

````ad-example
```ad-cpp
#include <iostream>  // for std::cout and std::cin

int main()
{
    std::cout << "Enter two numbers: ";

    int x{};
    std::cin >> x;

    int y{};
    std::cin >> y;

    std::cout << "You entered " << x << " and " << y << '\n';

    return 0;
}
```
This program inputs to two variables (this time as separate statements). We’ll run this program twice.

Run #1: When `std::cin >> x;` is encountered, the program will wait for input. Enter the value `4`. The input `4\n` goes into the input buffer, and the value `4` is extracted to variable `x`.

When `std::cin >> y;` is encountered, the program will again wait for input. Enter the value `5`. The input `5\n` goes into the input buffer, and the value `5` is extracted to variable `y`. Finally, the program will print `You entered 4 and 5`.

There should be nothing surprising about this run.

Run #2: When `std::cin >> x` is encountered, the program will wait for input. Enter `4 5`. The input `4 5\n` goes into the input buffer, but only the `4` is extracted to variable `x` (extraction stops at the space).

When `std::cin >> y` is encountered, the program will _not_ wait for input. Instead, the `5` that is still in the input buffer is extracted to variable `y`. The program then prints `You entered 4 and 5`.

Note that in run 2, the program didn’t wait for the user to enter additional input when extracting to variable `y` because there was already prior input in the input buffer that could be used.
````


```ad-note
Each line of input data in the input buffer is terminated by a `'\n'` character.
```

```ad-note
`std::cin` is buffered because it allows us to separate the entering of input from the extract of input. We can enter input once and then perform multiple extraction requests on it.
```

---

### Extracting whitespace characters
- Because extracting input ignores leading whitespace, this can lead to unexpected results when trying to extract whitespace characters to a char variable. One simple way to address this is to use the `std::cin.get()` function to perform the extraction instead, as this function does not ignore leading whitespace.

```cpp
#include <iostream>

int main()
{
    std::cout << "Input a keyboard character: "; // assume the user enters "a b" (without quotes)

    char ch{};
    std::cin >> ch; // extracts a, leaves " b\n" in stream
    std::cout << "You entered: " << ch << '\n';

    std::cin >> ch; // skips leading whitespace (the space), extracts b, leaves "\n" in stream
    std::cout << "You entered: " << ch << '\n';

    return 0;
}
```