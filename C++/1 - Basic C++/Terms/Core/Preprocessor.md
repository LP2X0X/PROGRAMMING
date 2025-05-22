---
tags: cpp, term, fundamental
---
- Prior to compilation, each code (.cpp) file goes through a **preprocessing** phase. In this phase, a program called the **preprocessor** makes various changes to the text of the code file. 
- The preprocessor does not actually modify the original code files in any way -- rather, *all changes made by the preprocessor happen either temporarily in-memory or using temporary files*.
- When the preprocessor has finished processing a code file, the result is called a **translation unit**. This translation unit is what is then compiled by the compiler.
</br>
- When the preprocessor runs, it scans through the code file (from top to bottom), looking for [[Preprocessor Directive|preprocessor directives]]. 

--- 

### Do not \#include .cpp file
- There are number of reasons for this:
	- Doing so can cause naming collisions between source files.
	- In a large project it can be hard to avoid one definition rules (ODR) issues.
	- Any change to such a .cpp file will cause both the .cpp file and any other .cpp file that includes it to recompile, which can take a long time. Headers tend to change less often than source files.
	- It is non-conventional to do so.