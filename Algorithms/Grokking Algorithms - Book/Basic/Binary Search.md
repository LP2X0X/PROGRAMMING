---
tags: algorithms, grokking, basic
---

### Summary
- Binary search is an algorithm; its input is a sorted list of elements (I’ll explain later why it needs to be sorted). 
	- If an element you’re looking for is in that list, binary search returns the position where it’s located.
	- Otherwise, binary search returns null.
- In general, for any list of n, binary search will take log<sub>2</sub>(n) steps to run in the worst case, whereas simple search will take n steps.