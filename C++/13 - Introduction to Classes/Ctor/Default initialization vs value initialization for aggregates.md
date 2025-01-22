```cpp
struct Fraction
{
	int numerator { }; // we should use { 0 } here, but for the sake of example we'll use value initialization instead
	int denominator { 1 };
};

int main()
{
	Fraction f1;          // f1.numerator value initialized to 0, f1.denominator defaulted to 1
	Fraction f2 {};       // f2.numerator value initialized to 0, f2.denominator defaulted to 1
	Fraction f3 { 6 };    // f3.numerator initialized to 6, f3.denominator defaulted to 1
	Fraction f4 { 5, 8 }; // f4.numerator initialized to 5, f4.denominator initialized to 8

	return 0;
}
```

- In the two lines from the above example:

```cpp
Fraction f1;          // f1.numerator value initialized to 0, f1.denominator defaulted to 1
Fraction f2 {};       // f2.numerator value initialized to 0, f2.denominator defaulted to 1
```

- You’ll note that `f1` is default initialized and `f2` is value initialized, yet the results are the same (`numerator` is initialized to `0` and `denominator` is initialized to `1`). So which should we prefer?

- The value initialization case (`f2`) is safer, because it will ensure any members with no default values are value initialized (and although we should always provide default values for members, this protects against the case where one is missed).

- Preferring value initialization has one more benefit -- it’s consistent with how we initialize objects of other types. Consistency helps prevent errors.

```ad-tip
For aggregates, prefer value initialization (with an empty braces initializer) to default initialization (with no braces).
```
