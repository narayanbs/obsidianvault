
# C++ Class / Struct  Implementation

 Let's  implement  a class that manages a **dynamic resource** (like raw memory on the heap).
 It will help us understand how memory management works in modern C++ (C++11 and beyond),

When a class manages a resource, it must implement the **Rule of 5**. If you write one of these, you probably need to write all five to avoid memory leaks, double-frees, or undefined behavior:

1. **Destructor**: Cleans up the resource.
    
2. **Copy Constructor**: Makes a deep copy of the resource.
    
3. **Copy Assignment Operator**: Cleans up the old resource and makes a deep copy of the new one.
    
4. **Move Constructor**: Steals the resource from a temporary object (zeroing out the old one).
    
5. **Move Assignment Operator**: Cleans up the old resource and steals the new one.
    
---
## The Complete Example: `DynamicArray`

Here is a complete, well-commented C++ class implementing all of these components.

C++

```cpp
#include <iostream>

struct DynamicArray {
 private:
  int *mdata;
  size_t msize;

 public:
  // standard constructor
  DynamicArray(size_t size) : msize(size) {
    mdata = new int[msize]{0};
    std::cout << "{Constructor} Allocated array of size " << msize << "\n";
  }

  // destructor
  ~DynamicArray() {
    delete[] mdata;
    std::cout << "{Destructor} Freed array of size " << msize << "\n";
  }

  // copy constructor 
  DynamicArray(const DynamicArray &other) : msize{other.msize} {
    mdata = new int[msize];
    for (size_t i = 0; i < msize; i++) {
      mdata[i] = other.mdata[i];
    }
    std::cout << "{Copy constructor} Deep copied size " << msize << "\n";
  }

  // copy assignment
  DynamicArray &operator=(const DynamicArray &other) {
    std::cout << "{Copy assignment} triggered" << "\n";
    if (this == &other) {
      return *this;
    }

    delete[] mdata;
    msize = other.msize;
    mdata = new int[msize];
    for (size_t i = 0; i < msize; i++) {
      mdata[i] = other.mdata[i];
    }
    return *this;
  }

  // move constructor
  DynamicArray(DynamicArray &&other) : mdata(other.mdata), msize(other.msize) noexcept {
    other.mdata = nullptr;
    other.msize = 0;
    std::cout << "{Move constructor} stole resource of size " << msize << "\n";
  }

  // move assignment  
  DynamicArray &operator=(DynamicArray &&other) noexcept {
    std::cout << "{Move assignment} triggered\n";
    if (this == &other) {
      return *this;
    }

    delete[] mdata;
    mdata = other.mdata;
    msize = other.msize;

    other.mdata = nullptr;
    other.msize = 0;

    return *this;
  }

  void set(size_t index, int value) {
    if (index < msize) mdata[index] = value;
  }
};
```

---

## How it works in practice (Main Function)

Let's see how C++ decides which constructor or assignment operator to call based on how you manipulate the objects.

C++

```cpp
int main() {
  std::cout << "----- Creating 'a' -------\n";
  DynamicArray a{5};
  a.set(0, 99);

  std::cout << "\n --- Copy constructor ('b' from 'a') --\n";
  DynamicArray b = a;

  std::cout << "\n ---- Copy assignment ('c' assigned to 'a') --\n";
  DynamicArray c{2};
  c = a;

  std::cout << "\n--- Move Constructor ('d' from moved 'b') ---\n";
  // std::move casts 'b' to an rvalue, forcing C++ to choose the Move Constructor
  DynamicArray d = std::move(b);
  // At this point, 'b' is empty and safe to destroy. 'd' now owns b's old memory.

  std::cout << "\n--- Move Assignment ('c' assigned to moved 'd') ---\n";
  // c frees its memory, steals d's memory. d becomes empty.
  c = std::move(d);
  // Note the value of the move expression is the reference returned
  // by the operator= function.

  std::cout << "\n--- Exiting Program (Destructors trigger) ---\n";
  return 0;
}
```

---
### Key Concepts to Remember

- **Copy vs. Move**: Copying duplicates the data on the heap (expensive). Moving just copies the _pointer_ address from one object to another and nulls out the original pointer (incredibly cheap, practically free).
    
- `noexcept`: Always mark your move constructors and move assignment operators as `noexcept`. This tells the C++ standard library (like `std::vector`) that it is safe to use your move operations during resizing without risking memory exceptions.
    
- **The Rvalue Reference (`&&`)**: The double ampersand signifies an "rvalue reference". It means the parameter is a temporary object or something explicitly marked with `std::move`, signaling that it's safe to rip its internals out.

----------
## Questions

### why do copy constructors take a const reference

This is one of the most fundamental design choices in C++, and it boils down to two critical reasons: **preventing an infinite compilation loop** and **allowing the constructor to accept temporary objects (rvalues)**.

If you tried to write a copy constructor without the `const` or without the reference, the compiler would either reject it or get stuck in a loop. Here is exactly why.

---

### 1. Why it *must* be a Reference: Preventing Infinite Recursion

To understand why it must be a reference, look at what happens if we try to pass by value instead:

```cpp
class MyClass {
public:
    // Hypothetical pass-by-value copy constructor (This will NOT compile)
    MyClass(MyClass other) { 
        // ... copy logic ...
    }
};

```

Remember how "pass-by-value" works in C++: **whenever you pass an object into a function by value, C++ makes a copy of it using its copy constructor.**

Do you see the trap?

1. You call the copy constructor to make a copy.
2. Because the argument `other` is passed by value, the program must *make a copy* of the argument before the function can even start.
3. To make that copy of the argument, it needs to call the copy constructor again.
4. To call *that* copy constructor, it must make another copy...

This creates an **infinite recursive loop** of copy constructor calls. To break this paradox, C++ requires you to pass the argument by reference (`&`). Passing by reference just passes an alias (an address) to the existing object, requiring absolutely no copies to be made.

---

### 2. Why it *must* be `const`: Accepting Temporaries (Rvalues)

Now that we know it has to be a reference, why does it need to be a `const` reference?

```cpp
MyClass(const MyClass& other);

```

As we explored previously, **a non-`const` lvalue reference (`MyClass&`) cannot bind to temporary objects (rvalues).** If you omit the `const`, your copy constructor will only work when you copy from a named variable. It will completely fail if you try to copy from a temporary object returned by a function or created on the fly.

#### The Failure Example:

```cpp
class MyClass {
public:
    MyClass(MyClass& other); // No const!
};

MyClass createObject() {
    MyClass temp;
    return temp; 
}

int main() {
    MyClass a;
    MyClass b = a;             // OK: 'a' is a named variable (lvalue)
    
    MyClass c = createObject(); // ERROR! The return value is a temporary (rvalue).
                                // A non-const reference cannot bind to it.
}

```

By adding `const`, you ensure the copy constructor can bind to **both** named variables and temporary objects.

---

### 3. The Side Benefit: Side-Effect Safety

There is also a philosophical reason. A copy constructor’s sole job is to read data from an existing object and replicate it into a new one. It has no business modifying the source object.

By marking the parameter as `const`, you tell the compiler to enforce this rule. If you accidentally try to alter the source object inside the copy constructor, the compiler will catch your mistake and throw an error.

---
### Why do move constructor/assignment accept non-const argument

This is the perfect logical next step. If copy operations use `const` references to protect the source object, move operations do the exact opposite: **they take a non-const rvalue reference (`T&&`) because they actively intend to modify and "steal" from the source object.**

If the argument were `const`, moving would be mechanically impossible. Here is exactly why move constructors and move assignment operators must take a non-const argument.

---