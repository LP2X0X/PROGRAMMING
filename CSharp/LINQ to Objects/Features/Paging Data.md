- If you need to get a certain number of records, you can use the Take()/TakeWhile()/TakeLast() and Skip()/SkipWhile()/SkipLast() methods.
	1.  Take() returns the specified number of records.
	2. TakeWhile() returns all of the records where the condition is true.
	3. TakeLast() returns the last specified numbers of records.

	1. Skip() passes over the specified number of records.
	2. SkipWhile() skips all the records where the condition is true.
	3. SkipLast() skips the last specified number of records.

```ad-info
These methods are exposed through the IEnumerable interface, so while you can’t use the paging operators on the LINQ statements directly, you can use them on the result of the LINQ statement.
```