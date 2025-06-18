---
tags: cpp, note
---

```ad-tip
Use parentheses to make it clear how a non-trivial compound expression should evaluate (even if they are technically unnecessary).
```

|Prec/Ass|Operator|Description|Pattern|
|---|---|---|---|
|1 L->R|::  <br>::|Global scope (unary)  <br>Namespace scope (binary)|::name  <br>class_name::member_name|
|2 L->R|()  <br>()  <br>type()  <br>type{}  <br>[]  <br>.  <br>->  <br>++  <br>––  <br>typeid  <br>const_cast  <br>dynamic_cast  <br>reinterpret_cast  <br>static_cast  <br>sizeof…  <br>noexcept  <br>alignof|Parentheses  <br>Function call  <br>Functional cast  <br>List init temporary object (C++11)  <br>Array subscript  <br>Member access from object  <br>Member access from object ptr  <br>Post-increment  <br>Post-decrement  <br>Run-time type information  <br>Cast away const  <br>Run-time type-checked cast  <br>Cast one type to another  <br>Compile-time type-checked cast  <br>Get parameter pack size  <br>Compile-time exception check  <br>Get type alignment|(expression)  <br>function_name(arguments)  <br>type(expression)  <br>type{expression}  <br>pointer[expression]  <br>object.member_name  <br>object_pointer->member_name  <br>lvalue++  <br>lvalue––  <br>typeid(type) or typeid(expression)  <br>const_cast<type>(expression)  <br>dynamic_cast<type>(expression)  <br>reinterpret_cast<type>(expression)  <br>static_cast<type>(expression)  <br>sizeof…(expression)  <br>noexcept(expression)  <br>alignof(type)|
|3 R->L|+  <br>-  <br>++  <br>––  <br>!  <br>not  <br>~  <br>(type)  <br>sizeof  <br>co_await  <br>&  <br>*  <br>new  <br>new[]  <br>delete  <br>delete[]|Unary plus  <br>Unary minus  <br>Pre-increment  <br>Pre-decrement  <br>Logical NOT  <br>Logical NOT  <br>Bitwise NOT  <br>C-style cast  <br>Size in bytes  <br>Await asynchronous call  <br>Address of  <br>Dereference  <br>Dynamic memory allocation  <br>Dynamic array allocation  <br>Dynamic memory deletion  <br>Dynamic array deletion|+expression  <br>-expression  <br>++lvalue  <br>––lvalue  <br>!expression  <br>not expression  <br>~expression  <br>(new_type)expression  <br>sizeof(type) or sizeof(expression)  <br>co_await expression (C++20)  <br>&lvalue  <br>*expression  <br>new type  <br>new type[expression]  <br>delete pointer  <br>delete[] pointer|
|4 L->R|->*  <br>.*|Member pointer selector  <br>Member object selector|object_pointer->*pointer_to_member  <br>object.*pointer_to_member|
|5 L->R|*  <br>/  <br>%|Multiplication  <br>Division  <br>Remainder|expression * expression  <br>expression / expression  <br>expression % expression|
|6 L->R|+  <br>-|Addition  <br>Subtraction|expression + expression  <br>expression - expression|
|7 L->R|<<  <br>>>|Bitwise shift left / Insertion  <br>Bitwise shift right / Extraction|expression << expression  <br>expression >> expression|
|8 L->R|<=>|Three-way comparison (C++20)|expression <=> expression|
|9 L->R|<  <br><=  <br>>  <br>>=|Comparison less than  <br>Comparison less than or equals  <br>Comparison greater than  <br>Comparison greater than or equals|expression < expression  <br>expression <= expression  <br>expression > expression  <br>expression >= expression|
|10 L->R|==  <br>!=|Equality  <br>Inequality|expression == expression  <br>expression != expression|
|11 L->R|&|Bitwise AND|expression & expression|
|12 L->R|^|Bitwise XOR|expression ^ expression|
|13 L->R|\||Bitwise OR|expression \| expression|
|14 L->R|&&  <br>and|Logical AND  <br>Logical AND|expression && expression  <br>expression and expression|
|15 L->R|\|  <br>or|Logical OR  <br>Logical OR|expression \| expression  <br>expression or expression|
|16 R->L|throw  <br>co_yield  <br>?:  <br>=  <br>*=  <br>/=  <br>%=  <br>+=  <br>-=  <br><<=  <br>>>=  <br>&=  <br>\|=  <br>^=|Throw expression  <br>Yield expression (C++20)  <br>Conditional  <br>Assignment  <br>Multiplication assignment  <br>Division assignment  <br>Remainder assignment  <br>Addition assignment  <br>Subtraction assignment  <br>Bitwise shift left assignment  <br>Bitwise shift right assignment  <br>Bitwise AND assignment  <br>Bitwise OR assignment  <br>Bitwise XOR assignment|throw expression  <br>co_yield expression  <br>expression ? expression : expression  <br>lvalue = expression  <br>lvalue *= expression  <br>lvalue /= expression  <br>lvalue %= expression  <br>lvalue += expression  <br>lvalue -= expression  <br>lvalue <<= expression  <br>lvalue >>= expression  <br>lvalue &= expression  <br>lvalue \|= expression  <br>lvalue ^= expression|
|17 L->R|,|Comma operator|expression, expression|