---
tags: cpp, term
---

- Programmers tend to make certain kinds of common mistakes, and some of those mistakes can be discovered by programs trained to look for them. These programs, generally known as **static analysis tools** (sometimes informally called _linters_) are programs that analyze your code to identify specific semantic issues (in this context, _static_ means that these tools analyze the source code).

- Some commonly recommended static analysis tools include:
	- Free:
		- [clang-tidy](https://clang.llvm.org/extra/clang-tidy/)
		- [cpplint](https://github.com/cpplint/cpplint)
		- [cppcheck](https://cppcheck.sourceforge.io/)
		- [SonarLint](https://www.sonarsource.com/open-source-editions/)
	
	- Most of these have extensions that allow them to integrate into your IDE. 
		- For example, [Clang Power Tools extension](https://marketplace.visualstudio.com/items?itemName=caphyon.ClangPowerTools).
	
	- Paid (may be free for Open Source projects):
		- [Coverity](https://www.synopsys.com/software-integrity/security-testing/static-analysis-sast.html)
		- [SonarQube](https://www.sonarsource.com/products/sonarqube/)