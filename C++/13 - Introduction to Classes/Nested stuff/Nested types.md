- Class types support another kind of member: **nested types** (also called **member types**). To create a nested type, you simply define the type inside the class, under the appropriate access specifier.

```cpp
#include <iostream>

class Fruit
{
public:
	// FruitType has been moved inside the class, under the public access specifier
        // We've also renamed it Type and made it an enum rather than an enum class
	enum Type
	{
		apple,
		banana,
		cherry
	};

private:
	Type m_type {};
	int m_percentageEaten { 0 };

public:
	Fruit(Type type) :
		m_type { type }
	{
	}

	Type getType() { return m_type;  }
	int getPercentageEaten() { return m_percentageEaten;  }

	bool isCherry() { return m_type == cherry; } // Inside members of Fruit, we no longer need to prefix enumerators with FruitType::
};

int main()
{
	// Note: Outside the class, we access the enumerators via the Fruit:: prefix now
	Fruit apple { Fruit::apple };

	if (apple.getType() == Fruit::apple)
		std::cout << "I am an apple";
	else
		std::cout << "I am not an apple";

	return 0;
}
```

- There are a few things worth pointing out here:
	- First, note that `FruitType` is now defined inside the class, where it has been renamed `Type` for reasons that we will discuss shortly.
	- Second, nested type `Type` has been defined at the top of the class. Nested type names must be fully defined before they can be used, so they are usually defined first.
	- Third, nested types follow normal access rules. `Type` is defined under the `public` access specifier, so that the type name and enumerators can be directly accessed by the public.
	- Fourth, class types act as a scope region for names declared within, just as namespaces do. Therefore the fully qualified name of `Type` is `Fruit::Type`, and the fully qualified name of the `apple` enumerator is `Fruit::apple`.
	- Finally, we changed our enumerated type from scoped to unscoped. Since the class itself is now acting as a scope region, it’s somewhat redundant to use a scoped enumerator as well. Changing to an unscoped enum means we can access enumerators as `Fruit::apple` rather than the longer `Fruit::Type::apple` we’d have to use if the enumerator were scoped.