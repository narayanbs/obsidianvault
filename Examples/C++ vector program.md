

```c++
#include <iostream>
#include <vector>

struct Person {
  std::string name;
  int age;

  explicit Person(std::string n, int a = 40) : name{n}, age{a} {}

  Person(const Person &p) : name{p.name}, age{p.age} {
    std::cout << "Copy constructor\n";
  }

  Person &operator=(const Person &p) = default;

  ~Person() {
    std::cout << "destructor\n";
  }
};

int main() {
  std::vector<Person> persons;
  persons.reserve(4);
  persons.push_back(Person{"one", 10});
  persons.push_back(Person{"two", 20});
  persons.push_back(Person{"three", 30});
  persons.push_back(Person{"four", 40});

  for (Person p : persons) {
    std::cout << p.name << " : " << p.age << "\n";
  }
}

```


The exact output is **not fully specified by the C++ standard**, because it depends on how `std::vector` grows its capacity (which varies by implementation). However, on most implementations (such as GCC/libstdc++ or Clang/libc++), you'll see something very similar to this.

### Step 1: `push_back` operations

Each statement like

```cpp
persons.push_back(Person{"one", 10});
```

does two things:

1. Creates a temporary `Person`.
    
2. Copies (or moves) it into the vector.
    

Your class has a user-defined copy constructor but **no move constructor**, so the vector must use the **copy constructor**.

Also, when the vector runs out of capacity, it reallocates and copies all existing elements into new storage.

A typical capacity growth is:

|Push|Capacity before|Reallocation?|Copies|
|---|---|---|---|
|one|0|yes (0→1)|1 (temporary)|
|two|1|yes (1→2)|2 (old element + temporary)|
|three|2|yes (2→4)|3 (2 old + temporary)|
|four|4|no|1 (temporary)|

So you'll likely see:

```
Copy constructor
destructor
Copy constructor
Copy constructor
destructor
destructor
Copy constructor
Copy constructor
Copy constructor
destructor
destructor
destructor
Copy constructor
destructor
```

The destructors correspond to:

- each temporary after `push_back`
    
- old elements destroyed after reallocation
    

---

### Step 2: Range-based for loop

```cpp
for (Person p : persons)
```

Notice that `p` is **by value**, not by reference.

Each iteration copies the element:

```
Copy constructor
one : 10
destructor
Copy constructor
two : 20
destructor
Copy constructor
three : 30
destructor
Copy constructor
four : 40
destructor
```

---

### Step 3: Vector destruction

When `main` ends, the vector destroys its four stored elements:

```
destructor
destructor
destructor
destructor
```

---

## Typical complete output

```
Copy constructor
destructor
Copy constructor
Copy constructor
destructor
destructor
Copy constructor
Copy constructor
Copy constructor
destructor
destructor
destructor
Copy constructor
destructor
Copy constructor
one : 10
destructor
Copy constructor
two : 20
destructor
Copy constructor
three : 30
destructor
Copy constructor
four : 40
destructor
destructor
destructor
destructor
destructor
```

---

### If you change the loop to

```cpp
for (const Person& p : persons)
```

there will be **no copy constructor calls** during iteration.

### If you also add a move constructor

```cpp
Person(Person&&) noexcept = default;
```

(or don't define the copy constructor yourself so the compiler generates moves), then `push_back` will mostly use moves instead of copies, significantly reducing the number of `"Copy constructor"` messages.


# Solution 

To reduce the copies and make it efficient, use this

```c++
#include <iostream>
#include <vector>

struct Person {
  std::string name;
  int age;

  explicit Person(std::string n, int a = 40) : name{n}, age{a} {}

  Person(const Person &p) : name{p.name}, age{p.age} {
    std::cout << "Copy constructor\n";
  }

  Person &operator=(const Person &p) = default;

  ~Person() {
    std::cout << "destructor\n";
  }
};

int main() {
  std::vector<Person> persons;
  persons.reserve(4);
  persons.emplace_back("one", 10);
  persons.emplace_back("two", 20);
  persons.emplace_back("three", 30);
  persons.emplace_back("four", 40);

  for (Person p : persons) {
    std::cout << p.name << " : " << p.age << "\n";
  }
}

```