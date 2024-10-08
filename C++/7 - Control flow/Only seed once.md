- Many PRNGs can be reseeded after the initial seeding. This essentially re-initializes the state of the random number generator, causing it to generate results starting from the new seed state. Reseeding should generally be avoided unless you have a specific reason to do so, as it can cause the results to be less random, or not random at all.

```ad-tip
Only seed a given pseudo-random number generator once, and do not reseed it.
```

- Here’s an example of a common mistake that new programmers make:

```cpp
#include <iostream>
#include <random>

int getCard()
{
    std::mt19937 mt{ std::random_device{}() }; // this gets created and seeded every time the function is called
    std::uniform_int_distribution card{ 1, 52 };
    return card(mt);
}

int main()
{
    std::cout << getCard() << '\n';

    return 0;
}
```

- In the `getCard()` function, the random number generator is being created and seeded every time the function is called. This is inefficient at best, and will likely cause poor random results.