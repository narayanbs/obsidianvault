Certainly! In C++, initializer lists are a powerful feature used to initialize data members of a class or structure. They can also be used to initialize base classes in case of inheritance. Here are some complex examples that demonstrate the use of initializer list constructors in various scenarios.

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

### 4. **Constructor with Default Arguments in Initializer List**
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
