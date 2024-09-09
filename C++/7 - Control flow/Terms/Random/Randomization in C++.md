- The randomization capabilities in C++ are accessible via the `<random>` header of the standard library. Within the random library, there are 6 PRNG families available for use (as of C++20):

|Type name|Family|Period|State size*|Performance|Quality|Should I use this?|
|---|---|---|---|---|---|---|
|minstd_rand  <br>minstd_rand0|Linear congruential generator|2^31|4 bytes|Bad|Awful|No|
|mt19937  <br>mt19937_64|Mersenne twister|2^19937|2500 bytes|Decent|Decent|Probably (see next section)|
|ranlux24  <br>ranlux48|Subtract and carry|10^171|96 bytes|Awful|Good|No|
|knuth_b|Shuffled linear congruential generator|2^31|1028 bytes|Awful|Bad|No|
|default_random_engine|Any of above (implementation defined)|Varies|Varies|?|?|No2|
|rand()|Linear congruential generator|2^31|4 bytes|Bad|Awful|Nono|