- An **access function** is a trivial public member function whose job is to retrieve or change the value of a private member variable.
- Access functions come in two flavors: getters and setters:
	- **Getters** (also sometimes called **accessors**) are public member functions that return the value of a private member variable. Getters are usually made const, so they can be called on both const and non-const objects.
	- **Setters** (also sometimes called **mutators**) are public member functions that set the value of a private member variable. Setters should be non-const, so they can modify the data members.

```cpp
#include <iostream>

class Date
{
private:
    int m_year { 2020 };
    int m_month { 10 };
    int m_day { 14 };

public:
    void print()
    {
        std::cout << m_year << '/' << m_month << '/' << m_day << '\n';
    }

    int getYear() const { return m_year; }        // getter for year
    void setYear(int year) { m_year = year; }     // setter for year

    int getMonth() const  { return m_month; }     // getter for month
    void setMonth(int month) { m_month = month; } // setter for month

    int getDay() const { return m_day; }          // getter for day
    void setDay(int day) { m_day = day; }         // setter for day
};

int main()
{
    Date d{};
    d.setYear(2021);
    std::cout << "The year is: " << d.getYear() << '\n';

    return 0;
}
```