---
tags:
  - cpp
  - distinguish
  - fundamental
---

- When we use **angled brackets**, we’re telling the preprocessor that this is a header file we didn’t write ourselves. The preprocessor will search for the header only in the directories specified by the `include directories`.
	- The `include directories` are configured as part of your project/IDE settings/compiler settings, and typically default to the directories containing the header files that come with your compiler and/or OS. The preprocessor will not search for the header file in your project’s source code directory.
- When we use **double-quotes**, we’re telling the preprocessor that this is a header file that we wrote. 
	- The preprocessor will first search for the header file in the current directory. If it can’t find a matching header there, it will then search the `include directories`.