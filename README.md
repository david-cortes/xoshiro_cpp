# Xoshiro Random Number generators

This is a C++ header-only wrapper for the Xoshiro256++ (64-bit) and Xhosiro128++ (32-bit) PRNG (pseudo-random number generators) from https://prng.di.unimi.it, which turns them into classes to be used as drop-in replacements for for C++'s standard random engines in the functions and classes from the `<random>` header.

_Note: it has a small change in the seeding logic compared to the original code._

# Why change the random engine

C++'s default random engine (at least in GCC and CLANG), while fast at generating random numbers, has some undesirable mathematical properties. When used for simulations and for initialization of randomized algorithms - particularly in statistical and machine learning models - oftentimes one can obtain consistently better results (e.g. statistical models that achieve slightly higher ROC AUC due to better initialization) from using better random number generators (see the author's webpage at the beginning for more detailed information).

These two RNGs (Xoshiro256++ and Xoshiro128++) are likely to produce better-quality random numbers when used for statistical models compared to linear congruential generators or mersenne-twister generators. They also have comparably good speed and storage size.

# License

Code is public domain (CC0) just like the original.

# Examples

To use this code, just copy the header file and include it (might require C++11 or higher). Has the same methods as the mersenne-twister engine from C++11.

```cpp
#include <iostream>
#include <iomanip>
#include <sstream>
#include <vector>
#include <random>
#include <algorithm>
#include "xoshiro.h"

int main()
{
    /* Initialize with some hard-coded state */
    Xoshiro::Xoshiro256PP rng;

    /* Can be seeded (also accepts a seed sequence) */
    rng = Xoshiro::Xoshiro256PP(1234);
    rng.seed(1234);

    /* Can burn out RNGs if desired */
    rng.discard(100);

    /* Now using it in place of std::default_random_engine */

    /* Shuffling a vector */
    std::vector<int> indices(10);
    std::iota(indices.begin(), indices.end(), 0);
    std::shuffle(indices.begin(), indices.end(), rng);
    std::cout << "Sample randomly-shuffled vector: [ ";
    for (auto val : indices)
        std::cout << val << " ";
    std::cout << "]" << std::endl;

    /* Generating normally-distributed numbers */
    std::cout << "Sample random numbers ~Normal(1, 3): [ ";
    std::normal_distribution<double> rnorm(1, 3);
    for (int ix = 0; ix < 10; ix++)
        std::cout << rnorm(rng) << " ";
    std::cout << "]" << std::endl;

    /* Can be serialized in the same way as standard generators */
    std::stringstream ss;
    ss << rng;
    std::cout << "\nNumbers generated right after serialization: [ ";
    for (int ix = 0; ix < 2; ix++)
        std::cout << rng() << " ";
    std::cout << "]" << std::endl;

    Xoshiro::Xoshiro256PP rng2;
    ss >> rng2;
    std::cout << "\nNumbers generated with de-serialized object: [ ";
    for (int ix = 0; ix < 2; ix++)
        std::cout << rng2() << " ";
    std::cout << "]" << std::endl;
    std::cout << "(should be the same as before)" << std::endl;

    /* Could also use the jumping functionality for parallel streams */
    Xoshiro::Xoshiro256PP rng_next = rng.jump();
    std::cout << std::setprecision(4) << std::fixed;
    std::uniform_real_distribution<double> runif(0, 1);
    std::cout << "\n(Jumping states)";
    std::cout << "\nNext random numbers (original): ";
    for (int ix = 0; ix < 5; ix++)
        std::cout << runif(rng) << " ";
    std::cout << "\nNext random numbers (parallel): ";
    for (int ix = 0; ix < 5; ix++)
        std::cout << runif(rng_next) << " ";
    std::cout << std::endl;
    return 0;
}
```
