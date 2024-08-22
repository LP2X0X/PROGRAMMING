- Another way to simplify things is to use a using-directive. A **using directive** allows _all_ identifiers in a given namespace to be referenced without qualification from the scope of the using-directive.

### Problem with using-directive
- In modern C++, using-directives generally offer little benefit (saving some typing) compared to the risk. This is due to two factors:
	1. Using-directives allow unqualified access to _all_ of the names from a namespace (potentially including lots of names you’ll never use).
	2. Using-directives do not prefer names from the namespace of the using-directive over other names.