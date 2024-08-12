- After you have established an unsafe context, you are then free to build pointers to data types using the  **\* operator and obtain the address of what is being pointed to using the & operator**. Unlike in C or C++, in C# the * operator is applied to the underlying type only, not as a prefix to each pointer variable name.

```csharp
// No! This is incorrect under C#!  
int *pi, *pj;  
// Yes! This is the way of C#.  
int* pi, pj;
```

### Example

```csharp
static unsafe void PrintValueAndAddress()  
{  
	int myInt;  
	// Define an int pointer, and  
	// assign it the address of myInt.  
	int* ptrToMyInt = &myInt;  
	// Assign value of myInt using pointer indirection.  
	*ptrToMyInt = 123;  
	// Print some stats.  
	Console.WriteLine("Value of myInt {0}", myInt);  
	Console.WriteLine("Address of myInt {0:X}", (int)&ptrToMyInt);  
}

## Result ## 
# **** Print Value And Address ****  
# Value of myInt 123  
# Address of myInt 90F7E698
```

---

- If you declare a pointer to a Point type, you will need to make use of the pointer field-access operator (represented by ->) to access its public members. In fact, using the pointer indirection operator (\*), it is possible to dereference a pointer to (once again) apply the dot operator notation.

```csharp
static unsafe void UsePointerToPoint()  
{  
	// Access members via pointer.  
	Point;  
	Point* p = &point;  
	p->x = 100;  
	p->y = 200;  
	Console.WriteLine(p->ToString());  

	// Access members via pointer indirection.  
	Point point2;  
	Point* p2 = &point2;  
	(*p2).x = 100;  
	(*p2).y = 200;  
	Console.WriteLine((*p2).ToString());  
}
```

---

