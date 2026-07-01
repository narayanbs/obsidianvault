# Random Numbers

In C++, there are two common ways to generate random numbers.

### Modern approach (recommended): `<random>`

This produces higher-quality random numbers than `rand()`.

```cpp
#include <iostream>
#include <random>

int main() {
    // Random number generator
    std::random_device rd;
    std::mt19937 gen(rd());

    // Generate integers between 1 and 100
    std::uniform_int_distribution<> dist(1, 100);

    for (int i = 0; i < 10; i++) {
        std::cout << dist(gen) << " ";
    }

    return 0;
}
```

Example output:

```
42 17 89 5 63 91 28 74 36 50
```

---

### Generate random floating-point numbers

```cpp
#include <iostream>
#include <random>

int main() {
    std::random_device rd;
    std::mt19937 gen(rd());

    // Random double between 0.0 and 1.0
    std::uniform_real_distribution<> dist(0.0, 1.0);

    std::cout << dist(gen) << std::endl;

    return 0;
}
```

Example output:

```
0.734829
```

---

### Older approach: `rand()` (not recommended)

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>

int main() {
    std::srand(std::time(nullptr)); // Seed the generator

    for (int i = 0; i < 10; i++) {
        std::cout << std::rand() % 100 + 1 << " "; // 1 to 100
    }

    return 0;
}
```

---

### Summary

| Method                      | C++ Standard | Quality | Recommended |
| --------------------------- | ------------ | ------- | ----------- |
| `std::mt19937` + `<random>` | C++11+       | High    | ✅ Yes       |
| `rand()`                    | C/C++        | Low     | ❌ No        |

For new C++ code, prefer the `<random>` library with `std::mt19937` and the appropriate distribution (`std::uniform_int_distribution`, `std::uniform_real_distribution`, etc.).

# Explanation of Code

 The modern C++ random library separates the **random number generator (engine)** from the **distribution**. This gives you much better randomness and flexibility than `rand()`.

Let's go through the code line by line.

```cpp
#include <random>
```

This includes the C++ random number library.

---

### Step 1: Create a source of randomness

```cpp
std::random_device rd;
```

`std::random_device` is used to obtain a random seed.

Think of it as asking the operating system for a random number.

For example,

```
rd() → 93847231
```

Every time you run the program, this value is likely to be different.

This seed is **not** used directly to generate all random numbers—it is only used to initialize the generator.

---

### Step 2: Create the random number engine

```cpp
std::mt19937 gen(rd());
```

This creates a random number generator called `gen`.

* `std::mt19937` is the **Mersenne Twister** engine.
* It is deterministic: given the same seed, it always produces the same sequence.

For example,

```
Seed = 5

Generator produces:

83
24
91
18
...
```

If you seed it with `5` tomorrow, you'll get exactly the same sequence.

Because we seeded it with `rd()`, the sequence is different each run.

You can also write

```cpp
std::mt19937 gen(42);
```

Now every execution gives the exact same random numbers, which is useful for debugging.

---

### Step 3: Specify the range

```cpp
std::uniform_int_distribution<> dist(1, 100);
```

The engine (`gen`) itself produces very large unsigned integers.

For example,

```
3499211612
581869302
3890346734
...
```

These aren't very useful if you only want numbers between 1 and 100.

The distribution converts those large numbers into your desired range.

Here,

```cpp
dist(1,100)
```

means

> Give every integer from **1 through 100** an equal probability.

---

### Step 4: Generate numbers

```cpp
dist(gen)
```

This is the most important line.

It works like this:

```
Generator
   |
   v
3499211612
   |
Distribution
   |
   v
57
```

So

```cpp
dist(gen)
```

means

1. Ask `gen` for a random value.
2. Transform it into the range 1–100.
3. Return the result.

---

### Inside the loop

```cpp
for (int i = 0; i < 10; i++) {
    std::cout << dist(gen) << " ";
}
```

Iteration by iteration:

```
gen -> huge number
      |
      v
dist -> 42

gen -> another huge number
      |
      v
dist -> 81

gen -> another huge number
      |
      v
dist -> 15
```

Output:

```
42 81 15 99 3 54 18 ...
```

---

## Why not just use `gen()`?

You can!

```cpp
std::cout << gen();
```

might print

```
3499211612
```

or

```
581869302
```

These are raw outputs from the engine.

Usually you want numbers in a specific range, so you use a distribution.

---

## Different distributions

The nice thing about the `<random>` library is that you can keep the same generator and simply change the distribution.

### Uniform integers

```cpp
std::uniform_int_distribution<> dist(1, 6);
```

Like rolling a die:

```
1 4 2 6 3 1
```

---

### Uniform real numbers

```cpp
std::uniform_real_distribution<> dist(0.0, 1.0);
```

Produces values like

```
0.273
0.814
0.091
```

---

### Normal (Gaussian) distribution

```cpp
std::normal_distribution<> dist(50, 10);
```

Produces values clustered around 50:

```
48
51
63
44
49
52
```

---

## Why separate the engine and distribution?

Because they have different responsibilities:

* **Engine (`std::mt19937`)**: Generates a sequence of pseudo-random bits.
* **Distribution (`std::uniform_int_distribution`, `std::uniform_real_distribution`, etc.)**: Maps those bits into values that follow a particular statistical distribution.

This separation lets you reuse the same engine with different distributions depending on what your program needs.

A useful analogy is:

* The **engine** is like a machine that produces an endless stream of random material.
* The **distribution** is like a mold that shapes that material into the form you want (integers in a range, floating-point values, bell-curve values, etc.).

