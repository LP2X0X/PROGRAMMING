Astute readers may note that the above examples don’t actually save any memory. 8 separate booleans values would normally take 8 bytes. But the examples above (using std::uint8_t) use 9 bytes -- 8 bytes to define the bit masks, and 1 byte for the flag variable!

Bit flags make the most sense when you have many identical flag variables. For example, in the example above, imagine that instead of having one person (me), you had 100. If you used 8 Booleans per person (one for each possible state), you’d use 800 bytes of memory. With bit flags, you’d use 8 bytes for the bit masks, and 100 bytes for the bit flag variables, for a total of 108 bytes of memory -- approximately 8 times less memory.

For most programs, the amount of memory saved using bit flags is not worth the added complexity. But in programs where there are tens of thousands or even millions of similar objects, using bit flags can reduce memory use substantially. It’s a useful optimization to have in your toolkit if you need it.

There’s another case where bit flags and bit masks can make sense. Imagine you had a function that could take any combination of 32 different options. One way to write that function would be to use 32 individual Boolean parameters:

```cpp
void someFunction(bool option1, bool option2, bool option3, bool option4, bool option5, bool option6, bool option7, bool option8, bool option9, bool option10, bool option11, bool option12, bool option13, bool option14, bool option15, bool option16, bool option17, bool option18, bool option19, bool option20, bool option21, bool option22, bool option23, bool option24, bool option25, bool option26, bool option27, bool option28, bool option29, bool option30, bool option31, bool option32);
```

Hopefully you’d give your parameters more descriptive names, but the point here is to show you how obnoxiously long the parameter list is.

Then when you wanted to call the function with options 10 and 32 set to true, you’d have to do so like this:

```cpp
someFunction(false, false, false, false, false, false, false, false, false, true, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, true);
```

This is ridiculously difficult to read (is that option 9, 10, or 11 that’s set to true?), and also means you have to remember which argument corresponds to which option (is setting the “edit flag” the 9th, 10th, or 11th parameter?).

Instead, if you defined the function using bit flags like this:

```cpp
void someFunction(std::bitset<32> options);
```

Then you could use bit flags to pass in only the options you wanted:

```cpp
someFunction(option10 | option32);
```

This is much more readable.

This is one of the reasons OpenGL, a well regarded 3d graphic library, opted to use bit flag parameters instead of many consecutive Boolean parameters.

Here’s a sample function call from OpenGL:

```cpp
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // clear the color and the depth buffer
```

GL_COLOR_BUFFER_BIT and GL_DEPTH_BUFFER_BIT are bit masks defined as follows (in gl2.h):

```cpp
#define GL_DEPTH_BUFFER_BIT               0x00000100
#define GL_STENCIL_BUFFER_BIT             0x00000400
#define GL_COLOR_BUFFER_BIT               0x00004000
```