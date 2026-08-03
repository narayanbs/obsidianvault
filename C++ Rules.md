The "Rule of X" guidelines in C++ are informal rules of thumb around **resource management, special member functions, and class design**. The most important ones are the **Rule of Zero, Three, Five, and Six**, plus a few related design rules.

---

## 1. Rule of Zero

**Prefer not to write any special member functions.**

If your class uses well-designed RAII types (like `std::string`, `std::vector`, `std::unique_ptr`, `std::shared_ptr`), let the compiler generate constructors, destructors, and assignment operators.

Example:

```cpp
class Person {
public:
    std::string name;
    std::vector<int> scores;
};
```

You do **not** write:

```cpp
Person();
Person(const Person&);
Person& operator=(const Person&);
Person(Person&&);
Person& operator=(Person&&);
~Person();
```

The compiler generates them correctly.

### Why?

The standard library types already manage their resources:

* `std::string` manages memory
* `std::vector` manages dynamic storage
* `std::unique_ptr` manages ownership

Your class simply composes these objects.

**Preferred modern C++ style:**

> Use existing RAII types and avoid manual resource management.

---

# 2. Rule of Three (C++98)

If a class requires one of these, it probably requires all three:

1. Destructor
2. Copy constructor
3. Copy assignment operator

The reason: a class managing a resource usually needs to define how copying works.

Example:

```cpp
class Buffer {
    int* data;

public:
    Buffer(size_t size)
        : data(new int[size]) {}

    ~Buffer() {
        delete[] data;
    }

    Buffer(const Buffer& other) {
        // deep copy
        data = new int[...];
    }

    Buffer& operator=(const Buffer& other) {
        // deep copy assignment
        return *this;
    }
};
```

Without custom copy operations:

```cpp
Buffer a(100);
Buffer b = a;
```

would copy only the pointer:

```
a.data ---> memory
b.data ---^
```

Both objects would try to delete the same memory.

---

# 3. Rule of Five (C++11)

Move semantics added two more special member functions:

4. Move constructor
5. Move assignment operator

So if you need one of the five, you probably need all five.

```cpp
class Buffer {
public:
    ~Buffer();

    Buffer(const Buffer&);
    Buffer& operator=(const Buffer&);

    Buffer(Buffer&&);
    Buffer& operator=(Buffer&&);
};
```

Example move constructor:

```cpp
Buffer(Buffer&& other)
{
    data = other.data;
    other.data = nullptr;
}
```

Instead of copying:

```
old object:
data ---> large memory block

new object:
data ---> same memory block
```

Ownership is transferred.

---

# 4. Rule of Six (C++11 and later)

Sometimes people add the default constructor as the sixth function:

1. Default constructor
2. Destructor
3. Copy constructor
4. Copy assignment
5. Move constructor
6. Move assignment

Example:

```cpp
class Resource {
public:
    Resource();

    ~Resource();

    Resource(const Resource&);
    Resource& operator=(const Resource&);

    Resource(Resource&&);
    Resource& operator=(Resource&&);
};
```

This is less of a formal rule and more of a teaching extension of the Rule of Five.

---

# 5. Rule of Five Zero

A common modern recommendation:

> If you need the Rule of Five, consider redesigning until you need the Rule of Zero.

Example:

### Bad:

```cpp
class File {
    FILE* handle;

public:
    ~File() {
        fclose(handle);
    }
};
```

You now need:

* destructor
* copy control
* move control

### Better:

```cpp
class File {
    std::unique_ptr<FILE, FileCloser> handle;
};
```

Now:

```cpp
File f1;
File f2 = std::move(f1);
```

works automatically.

---

# 6. Rule of One

A class should have **one responsibility**.

Bad:

```cpp
class Employee {
public:
    void saveToDatabase();
    void calculateSalary();
    void sendEmail();
};
```

This class has three jobs.

Better:

```cpp
class Employee {};
class EmployeeRepository {};
class PayrollService {};
class EmailService {};
```

This is the Single Responsibility Principle (SRP).

---

# 7. Rule of Least Privilege

Give the minimum access required.

Avoid:

```cpp
class Account {
public:
    double balance;
};
```

Anyone can modify it:

```cpp
account.balance = -100000;
```

Better:

```cpp
class Account {
private:
    double balance;

public:
    double getBalance() const;
    void deposit(double amount);
};
```

---

# 8. Rule of Prefer `const`

Make things immutable whenever possible.

Instead of:

```cpp
int getValue();
```

prefer:

```cpp
int getValue() const;
```

Meaning:

```cpp
const Object obj;
obj.getValue(); // works
```

---

# 9. Rule of RAII (Resource Acquisition Is Initialization)

One of the most important C++ concepts.

Resources belong to objects.

Instead of:

```cpp
openFile();

try {
    process();
}
catch (...) {
    closeFile();
}
closeFile();
```

Use:

```cpp
class File {
public:
    File() {
        openFile();
    }

    ~File() {
        closeFile();
    }
};
```

Now:

```cpp
{
    File f;
    process();
}
```

The destructor automatically releases the resource.

RAII applies to:

* memory
* files
* sockets
* mutexes
* database connections

---

# 10. Rule of No Raw `new`/`delete`

Modern C++:

Avoid:

```cpp
int* p = new int(5);
delete p;
```

Prefer:

```cpp
auto p = std::make_unique<int>(5);
```

or:

```cpp
auto p = std::make_shared<int>(5);
```

The compiler and standard library handle lifetime.

---

# Quick reference table

| Rule                        | Meaning                                                     |
| --------------------------- | ----------------------------------------------------------- |
| Rule of Zero                | Write no special members; use RAII types                    |
| Rule of Three               | Destructor + copy constructor + copy assignment go together |
| Rule of Five                | Add move constructor + move assignment                      |
| Rule of Six                 | Add default constructor as a sixth function                 |
| Rule of One                 | One class, one responsibility                               |
| Rule of RAII                | Manage resources through object lifetime                    |
| Rule of `const`             | Make functions/data immutable when possible                 |
| Rule of No Raw `new/delete` | Prefer smart pointers and containers                        |

---

For modern C++ (C++17/C++20/C++23), the hierarchy of preference is usually:

1. **Rule of Zero** ✅
2. If impossible, use **Rule of Five**
3. If you manually manage resources, make ownership and copying/moving explicit
4. Avoid raw resource ownership whenever possible

This single mindset eliminates a large fraction of C++ memory and lifetime bugs.
