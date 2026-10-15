	
# C++ Constructor Initialization

When you build a constructor, you have two choices for how to give your variables their starting values: **Assignment** (the naive way) or **Initialization** (the professional way).

Here is exactly how they look side-by-side in code:

C++

```cpp
#include <string>

class Wizard {
private:
    std::string name;
    int mana;

public:
    // Approach A: Assignment inside the body (Avoid this)
    Wizard(std::string n, int m) {
        name = n;   // Triggers an extra, useless setup step
        mana = m;
    }

    // Approach B: Member Initializer List (Do this!)
    // Notice the colon (:) and the brace initialization {}
    Wizard(std::string n, int m) : name{n}, mana{m} {
        // The body stays completely empty!
    }
};
```

### Why Approach B is Superior

To understand why the **member initializer list** is so important, you have to look at the "hidden" timeline of how C++ creates an object.

When you instantiate an object, C++ allocates memory and sets up your member variables _before_ it ever executes the first line of code inside the curly braces `{}` of your constructor body.

- **With Approach A (Body Assignment):** C++ creates `name` as an empty string (`""`). Then, it enters the constructor body, throws away that empty string, and overwrites it with `n`. You've essentially initialized it twice.
    
- **With Approach B (Initializer List):** C++ creates `name` directly using `n`. No wasted steps, no temporary variables thrown away. It is pure efficiency.
  
---
### Three Times You _Must_ Use Member initializer list

While it's a great performance habit for standard variables, there are three scenarios where C++ will flat-out refuse to compile your code if you don't use a member initializer list:

#### 1. Constant Members (`const`)

Constants cannot be changed after they are created. If you try to assign them inside the constructor body, the compiler will treat it as trying to mutate a constant.

C++

```cpp
class Item {
    const int id;
    // Item(int new_id) { id = new_id; } // ERROR! id is const.
    Item(int new_id) : id{new_id} {}    // Perfectly legal.
};
```

#### 2. References (`&`)

References in C++ must be bound to an object the exact moment they are created; they cannot be "empty" and bound later.

C++

```cpp
class Logger {
    Window& output_window;
    // Must be bound immediately using the initializer list:
    Logger(Window& w) : output_window{w} {} 
};
```

#### 3. Classes Without a Default Constructor

If your class contains another object that _requires_ arguments to be created, you must supply those arguments in the initializer list.

---

### A Sneaky Gotcha: Order Matters!

There is one trap to watch out for. C++ always initializes member variables in the order they are **declared in the class definition**, _not_ the order you list them in your initializer list.

C++

```cpp
class BadExample {
    int x;
    int y;
public: 
    // x is declared first, so x will initialize first using an uninitialized 'y'!
    BadExample(int val) : y{val}, x{y + 5} {} 
};
```

> **Pro-Tip:** To avoid chaotic bugs, always list your variables in the member initializer list in the **exact same top-to-bottom order** they are declared inside your class. Most modern compilers will give you a warning if you mix them up.

#### More Examples 

In C++, member initializer lists can also be used to initialize base classes in case of inheritance. Here are some complex examples that demonstrate the use of initializer list constructors in various scenarios.

### 1. **Basic Example with Multiple Data Members**
This example shows how to initialize multiple data members in the constructor using the initializer list.

```cpp
#include <iostream>
#include <string>
#include <vector>

class Student {
private:
    std::string name;
    int age;
    std::vector<int> grades;

public:
    // Constructor using initializer list
    Student(std::string n, int a, std::vector<int> g)
        : name(n), age(a), grades(g) {}

    void display() {
        std::cout << "Name: " << name << "\nAge: " << age << "\nGrades: ";
        for (int grade : grades) {
			std::cout << grade << " ";
        }
        std::cout << std::endl;
    }
};

int main() {
    std::vector<int> grades = {90, 85, 88};
    Student student("John", 20, grades);
    student.display();
    return 0;
}
```

### 2. **Constructor with Inherited Class and Initializing Base Class**
This example demonstrates how the initializer list is used to call a base class constructor from a derived class constructor.

```cpp
#include <iostream>

class Base {
public:
    int baseValue;

    // Base class constructor
    Base(int val) : baseValue(val) {
        std::cout << "Base class constructor called with value " << val << std::endl;
    }
};

class Derived : public Base {
public:
    int derivedValue;

    // Derived class constructor
    Derived(int val1, int val2) 
        : Base(val1), derivedValue(val2) { // Initializing base class in initializer list
        std::cout << "Derived class constructor called with value " << val2 << std::endl;
    }

    void display() {
        std::cout << "Base Value: " << baseValue << ", Derived Value: " << derivedValue << std::endl;
    }
};

int main() {
    Derived obj(5, 10);
    obj.display();
    return 0;
}
```

### 3. **Constructor with Constant Member Variables**
In this example, the class contains constant members, and their initialization must be done via the initializer list.

```cpp
#include <iostream>

class MyClass {
private:
    const int id;
    const double salary;

public:
    // Constructor using initializer list to initialize constant members
    MyClass(int i, double s) : id(i), salary(s) {}

    void display() {
        std::cout << "ID: " << id << ", Salary: " << salary << std::endl;
    }
};

int main() {
    MyClass obj(1001, 50000.0);
    obj.display();
    return 0;
}
```

### 4. **Constructor with Default Arguments in member Initializer List**
You can also provide default arguments in the constructor to initialize the members with default values if not explicitly passed.

```cpp
#include <iostream>

class Rectangle {
private:
    int width, height;

public:
    // Constructor with default values in the initializer list
    Rectangle(int w = 5, int h = 10) : width(w), height(h) {}

    void display() {
        std::cout << "Width: " << width << ", Height: " << height << std::endl;
    }
};

int main() {
    Rectangle rect1;         // Uses default values
    Rectangle rect2(15, 20); // Uses passed values
    rect1.display();
    rect2.display();
    return 0;
}
```

### 5. **Constructor with a Member Object**
This example demonstrates a constructor with a member object that is itself initialized via the initializer list.

```cpp
#include <iostream>

class Engine {
private:
    int horsepower;

public:
    Engine(int hp) : horsepower(hp) {}

    void display() {
        std::cout << "Horsepower: " << horsepower << std::endl;
    }
};

class Car {
private:
    Engine engine;
    std::string model;

public:
    // Constructor with member object initialization
    Car(int hp, const std::string &m) : engine(hp), model(m) {}

    void display() {
        std::cout << "Car Model: " << model << std::endl;
        engine.display();
    }
};

int main() {
    Car car(300, "Tesla Model S");
    car.display();
    return 0;
}
```

### 6. **Multiple Inheritance with Multiple Base Classes**
In this case, the initializer list initializes multiple base classes when a derived class inherits from more than one base class.

```cpp
#include <iostream>

class Engine {
public:
    Engine() {
        std::cout << "Engine initialized." << std::endl;
    }
};

class Wheels {
public:
    Wheels() {
        std::cout << "Wheels initialized." << std::endl;
    }
};

class Car : public Engine, public Wheels {
public:
    Car() : Engine(), Wheels() {
        std::cout << "Car initialized." << std::endl;
    }
};

int main() {
    Car car;
    return 0;
}
```

### 7. **Lambda Function in Constructor**
You can also use lambda functions directly in the initializer list. This is useful for more complex member initializations.

```cpp
#include <iostream>
#include <functional>

class MyClass {
private:
    int value;
    std::function<int()> initializer;

public:
    // Constructor with lambda function
    MyClass(int val) : value(val), initializer([=]() { return val * 2; }) {}

    void display() {
        std::cout << "Value: " << value << ", Initialized Value: " << initializer() << std::endl;
    }
};

int main() {
    MyClass obj(5);
    obj.display();
    return 0;
}
```

These examples show various scenarios where initializer lists are used in C++. From initializing data members and constants to handling inheritance and more complex situations like lambdas, initializer lists provide an efficient and clear way to initialize your objects.


## Constructor argument best practices

When it comes to passing arguments to C++ constructors, the "best practice" isn't a one-size-fits-all rule. It depends entirely on **ownership**, **modern C++ standards (C++11 and beyond)**, and **performance**.

Here is the breakdown of the industry-standard best practices for constructor arguments.

---

## 1. The Big Three Rules of Thumb

How you pass an argument depends on what the constructor does with it:

### A. Pass by Value and `std::move` (For Cheap-to-Move or Sinks)

If your constructor is going to take ownership of the argument and store a copy of it in a member variable (a "sink"), **pass it by value and move it**.

This works brilliantly for types with move semantics (like `std::string`, `std::vector`, or smart pointers). It results in at most one move operation, which is incredibly cheap.

```cpp
#include <string>
#include <utility>

class User {
private:
    std::string m_name; // Member variable

public:
    // Pass by value, then move into the member
    explicit User(std::string name) 
        : m_name(std::move(name)) {} 
};

```

### B. Pass by Reference-to-Const (For Expensive-to-Copy, Non-Movable, or Read-Only)

If the constructor needs to read the data but *doesn't* take ownership (e.g., storing a reference/pointer or just inspecting the data during construction), or if you are working with legacy code without move semantics.

```cpp
class ConfigRenderer {
private:
    const LargeConfigData& m_config; // Just holding a reference

public:
    // Pass by const reference to avoid copying
    explicit ConfigRenderer(const LargeConfigData& config) 
        : m_config(config) {}
};

```

### C. Pass by Trivial Value (For Built-in / Small Types)

For cheap primitives (like `int`, `double`, `bool`, `char`, or small enums/structs), always pass by value. Passing these by reference actually hurts performance because it introduces pointer indirection.

```cpp
class Point {
private:
    double m_x, m_y;

public:
    // Trivial types are cheap to copy
    Point(double x, double y) : m_x(x), m_y(y) {}
};

```

---

## 2. Mark Single-Argument Constructors as `explicit`

By default, C++ allows implicit conversions. If you have a constructor that takes a single argument, the compiler will happily convert that argument type into your class type without telling you, which often leads to bugs.

```cpp
class Wallet {
private:
    int m_dollars;
public:
    // Prevent implicit conversion from int to Wallet
    explicit Wallet(int dollars) : m_dollars(dollars) {}
};

void printWallet(const Wallet& w);

// Usage:
printWallet(42); // ❌ Compile error if explicit (Good!). Succeeds if not explicit (Confusing!).
printWallet(Wallet(42)); //  Explicit and clear.

```

---

## 3. Smart Pointers and Dependency Injection

If your constructor requires a dependency (like a database connection or a logger):

* **Use `std::unique_ptr` by value** if the class is taking *exclusive* ownership of that resource.
* **Use `std::shared_ptr` by value** if the class is going to *share* ownership of that resource.

```cpp
#include <memory>

class Controller {
private:
    std::unique_ptr<Database> m_db;

public:
    // Passing unique_ptr by value forces the caller to use std::move, 
    // making the transfer of ownership explicitly clear.
    explicit Controller(std::unique_ptr<Database> db) 
        : m_db(std::move(db)) {}
};

```

---

## Explicit  keyword -- More details

In C++, the `explicit` keyword is used on constructors (and conversion operators since C++11) to **prevent unintended implicit conversions**.

### Without `explicit`

Consider this class:

```cpp
#include <iostream>
using namespace std;

class Distance {
public:
    Distance(int meters) {
        cout << "Distance: " << meters << " meters\n";
    }
};

void printDistance(Distance d) {
    cout << "Printing distance\n";
}

int main() {
    Distance d = 10;      // Implicit conversion from int to Distance
    printDistance(20);    // Also implicitly converts int to Distance
}
```

**Output:**

```
Distance: 10 meters
Distance: 20 meters
Printing distance
```

Here, the compiler automatically converts an `int` into a `Distance` object because the constructor takes a single argument.

---

### With `explicit`

```cpp
#include <iostream>
using namespace std;

class Distance {
public:
    explicit Distance(int meters) {
        cout << "Distance: " << meters << " meters\n";
    }
};

void printDistance(Distance d) {}

int main() {
    Distance d = 10;      // ❌ Error
    printDistance(20);    // ❌ Error

    Distance d2(10);      // ✅ OK
    Distance d3{20};      // ✅ OK
    printDistance(Distance(30)); // ✅ OK
}
```

Now, the compiler **requires you to explicitly create the object** instead of silently converting an `int`.

---

## Why is this useful?

Suppose you have:

```cpp
class String {
public:
    explicit String(int size);
};
```

Without `explicit`, this would compile:

```cpp
String s = 100;
```

But what does `100` mean?

* A string with capacity 100?
* A string containing `"100"`?
* Something else?

Making the constructor `explicit` forces the programmer to write:

```cpp
String s(100);
```

which makes the intention clear.

---

## Rule of thumb

Use `explicit` for **single-parameter constructors** (or constructors that can be called with one argument due to default parameters) unless you intentionally want implicit conversions.

```cpp
class A {
public:
    explicit A(int x);      // Good practice
};

class B {
public:
    B(int x);               // Allows implicit conversion
};
```

---

## Constructors affected by `explicit`

It applies to constructors that can be used as conversion constructors:

```cpp
class Example {
public:
    explicit Example(int x);
    explicit Example(int x, int y = 0); // Also behaves like a single-argument constructor
};
```
