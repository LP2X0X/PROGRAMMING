### There are five basic types of numeric conversions.
1. Converting an integral type to any other integral type (excluding integral promotions):

```cpp
short s = 3; // convert int to short
long l = 3; // convert int to long
char ch = s; // convert short to char
unsigned int u = 3; // convert int to unsigned int
```

2. Converting a floating point type to any other floating point type (excluding floating point promotions):

```cpp
float f = 3.0; // convert double to float
long double ld = 3.0; // convert double to long double
```

3. Converting a floating point type to any integral type:

```cpp
int i = 3.5; // convert double to int
```

4. Converting an integral type to any floating point type:

```cpp
double d = 3; // convert int to double
```

5. Converting an integral type or a floating point type to a bool:

```cpp
bool b1 = 3; // convert int to bool
bool b2 = 3.0; // convert double to bool
```