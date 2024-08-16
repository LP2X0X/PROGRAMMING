- When outputting floating point numbers, `std::cout` has a default precision of 6 -- that is, it assumes all floating point variables are only significant to 6 digits (the minimum precision of a float), and hence it will truncate anything after that.
- We can override the default precision that std::cout shows by using an `output manipulator` function named `std::setprecision()`.
- **Output manipulators** alter how data is output, and are defined in the _iomanip_ header.

```ad-tip
Output manipulators (and input manipulators) are sticky -- meaning if you set them, they will remain set.
The one exception is `std::setw`. Some IO operations reset `std::setw`, so `std::setw` should be used every time it is needed.
```
