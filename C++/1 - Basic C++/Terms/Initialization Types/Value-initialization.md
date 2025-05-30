---
tags: cpp, term, fundamental
---
- This is the initialization performed when an object is constructed with **an empty** [[Initializer|initializer]].

```cpp
#include <cassert>
#include <iostream>
#include <string>
#include <vector>
 
struct T1
{
    int mem1;
    std::string mem2;
    virtual void foo() {} // make sure T1 is not an aggregate
}; // implicit default constructor
 
struct T2
{
    int mem1;
    std::string mem2;
    T2(const T2&) {} // user-provided copy constructor
};                   // no default constructor
 
struct T3
{
    int mem1;
    std::string mem2;
    T3() {} // user-provided default constructor
};
 
std::string s{}; // class => default-initialization, the value is ""
 
int main()
{
    int n{};                // scalar => zero-initialization, the value is 0
    assert(n == 0);
    double f = double();    // scalar => zero-initialization, the value is 0.0
    assert(f == 0.0);
    int* a = new int[10](); // array => value-initialization of each element
    assert(a[9] == 0);      //          the value of each element is 0
    T1 t1{};                // class with implicit default constructor =>
    assert(t1.mem1 == 0);   //     t1.mem1 is zero-initialized, the value is 0
    assert(t1.mem2 == "");  //     t1.mem2 is default-initialized, the value is ""
//  T2 t2{};                // error: class with no default constructor
    T3 t3{};                // class with user-provided default constructor =>
    std::cout << t3.mem1;   //     t3.mem1 is default-initialized to indeterminate value
    assert(t3.mem2 == "");  //     t3.mem2 is default-initialized, the value is ""
    std::vector<int> v(3);  // value-initialization of each element
    assert(v[2] == 0);      // the value of each element is 0
    std::cout << '\n';
    delete[] a;
}
```