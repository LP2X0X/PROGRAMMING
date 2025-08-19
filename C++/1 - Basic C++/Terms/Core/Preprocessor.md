---
tags:
  - cpp
  - term
  - fundamental
---
- Prior to compilation, each code (.cpp) file goes through a **preprocessing** phase. In this phase, a program called the **preprocessor** makes various changes to the text of the code file. 
- The preprocessor does not actually modify the original code files in any way -- rather, *all changes made by the preprocessor happen either temporarily in-memory or using temporary files*.
- When the preprocessor has finished processing a code file, the result is called a **translation unit**. This translation unit is what is then compiled by the compiler.
</br>
- When the preprocessor runs, it scans through the code file (from top to bottom), looking for [[Preprocessor Directive|preprocessor directives]]. 
