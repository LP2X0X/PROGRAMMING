- In C++, unconditional jumps are implemented via a **goto statement**, and the spot to jump to is identified through use of a **[[Statement label|statement label]]**.

```ad-important
Best practice:
Avoid goto statements (unless the alternatives are significantly worse for code readability). One notable exception is when you need to exit a nested loop but not the entire function -- in such a case, a goto to just beyond the loops is probably the cleanest solution.
```