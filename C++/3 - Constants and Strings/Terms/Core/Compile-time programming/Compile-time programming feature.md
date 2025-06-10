---
tags: cpp, term, fundamental
---

- While the [[The as-if rule|as-if rule]] is great for improving performance, it leaves us reliant on the sophistication of the compiler to actually determine what can evaluate at compile-time.
- To improve upon this situation, the C++ language provides ways for us to be explicit about what parts of code we want to execute at compile-time. The use of language features that result in compile-time evaluation is called **compile-time programming**.
- To summarize, compile-time evaluation allows us to write programs that are both more performant and of higher quality (more secure and less buggy)!