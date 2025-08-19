---
tags:
  - cpp
  - term
  - fundamental
---

- To maximize the chance that missing includes will be flagged by compiler, order your \#includes as follows (skipping any that are not relevant):
	- The paired header file for this code file (e.g. `add.cpp` should `#include "add.h"`)
	- Other headers from the same project (e.g. `#include "mymath.h"`)
	- 3rd party library headers (e.g. `#include <boost/tuple/tuple.hpp>`)
	- Standard library headers (e.g. `#include <iostream>`)
- The headers for each grouping should be sorted alphabetically (unless the documentation for a 3rd party library instructs you to do otherwise).
```ad-tip
This way, if one of your user-defined headers is missing an #include for a 3rd party library or standard library header, it’s more likely to cause a compile error so you can fix it.
```