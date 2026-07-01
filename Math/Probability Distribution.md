Yes. These **distributions are a core part of probability theory**.

In general, a **probability distribution** describes how likely different outcomes of a random variable are.

A random variable is just a variable whose value comes from a random process.

Example:

```text
Roll a dice

Random variable X = the number rolled

Possible values:
X = 1, 2, 3, 4, 5, 6
```

The distribution tells you the probability of each value:

[
P(X=1)=\frac16
]

[
P(X=2)=\frac16
]

and so on.

That is a **uniform distribution** because every outcome is equally likely.

genui{"probability_statistics_learning_block":{"type_id":"PROBABILITY_INTERSECTION"}}

---

### In probability, distributions are models of randomness

Different situations have different patterns.

### 1. Uniform distribution

"Everything is equally likely."

Example:

Rolling a fair die:

```
1  2  3  4  5  6
|  |  |  |  |  |
equal probability
```

C++:

```cpp
std::uniform_int_distribution<int> dice(1,6);
```

---

### 2. Normal distribution

"The middle is common, extremes are rare."

Example: human heights

```
          *
        *****
      *********
    *************
  *****************
--------------------
short       tall
```

Most people are around the average height.

C++:

```cpp
std::normal_distribution<double> height(170,10);
```

Meaning:

* mean = 170 cm
* standard deviation = 10 cm

---

### 3. Bernoulli distribution

A yes/no outcome.

Example:

```
Coin flip:

Heads  → probability 0.5
Tails  → probability 0.5
```

C++:

```cpp
std::bernoulli_distribution coin(0.5);
```

---

### 4. Poisson distribution

"How many times does something happen in a fixed interval?"

Example:

Number of customers arriving in an hour:

```
0 customers
1 customer
2 customers
3 customers
...
```

C++:

```cpp
std::poisson_distribution<int> customers(5);
```

---

So the connection is:

```
Probability theory
        |
        |
  Probability distributions
        |
        |
  Mathematical models of random events
        |
        |
  C++ <random> library implements them
```

The C++ classes like:

```cpp
std::normal_distribution
std::uniform_int_distribution
std::bernoulli_distribution
```

are basically **computer versions of mathematical probability distributions**. They let you simulate situations where randomness follows a known probability pattern.
