Inheritance specifiers (`public`, `protected`, `private`) control **how the base class's public and protected members are viewed from the derived class and from outside code**.

Consider:

```cpp
class Base {
public:
    int pub;

protected:
    int prot;

private:
    int priv;
};
```

and

```cpp
class Derived : ??? Base {
};
```

---

## 1. Public inheritance

```cpp
class Derived : public Base {
};
```

Mapping:

| Base member | Becomes in Derived |
| ----------- | ------------------ |
| public      | public             |
| protected   | protected          |
| private     | inaccessible       |

Example:

```cpp
Derived d;

d.pub = 10;    // OK
// d.prot = 20; // Error
```

Inside `Derived`:

```cpp
class Derived : public Base {
public:
    void f() {
        pub = 1;    // OK
        prot = 2;   // OK
        // priv = 3; // Error
    }
};
```

### Meaning

Public inheritance models an **"is-a"** relationship.

```cpp
class Animal {};
class Dog : public Animal {};
```

A `Dog` is an `Animal`.

Therefore:

```cpp
Dog d;
Animal* a = &d; // OK
```

This is the most common form of inheritance.

---

## 2. Protected inheritance

```cpp
class Derived : protected Base {
};
```

Mapping:

| Base member | Becomes in Derived |
| ----------- | ------------------ |
| public      | protected          |
| protected   | protected          |
| private     | inaccessible       |

Example:

```cpp
Derived d;

// d.pub = 10; // Error
```

Inside `Derived`:

```cpp
class Derived : protected Base {
public:
    void f() {
        pub = 1;    // OK
        prot = 2;   // OK
    }
};
```

Further derived classes can still access them:

```cpp
class Child : public Derived {
public:
    void g() {
        pub = 1;   // OK
        prot = 2;  // OK
    }
};
```

### Meaning

The implementation of `Base` is available to `Derived` and its descendants, but hidden from outside users.

---

## 3. Private inheritance

```cpp
class Derived : private Base {
};
```

Mapping:

| Base member | Becomes in Derived |
| ----------- | ------------------ |
| public      | private            |
| protected   | private            |
| private     | inaccessible       |

Example:

```cpp
Derived d;

// d.pub = 10; // Error
```

Inside `Derived`:

```cpp
class Derived : private Base {
public:
    void f() {
        pub = 1;    // OK
        prot = 2;   // OK
    }
};
```

But further derived classes cannot access them:

```cpp
class Child : public Derived {
public:
    void g() {
        // pub = 1;   // Error
        // prot = 2;  // Error
    }
};
```

### Meaning

Private inheritance is closer to **"implemented in terms of"** than **"is-a"**.

```cpp
class Stack : private std::vector<int> {
};
```

Users of `Stack` cannot treat it as a `vector`.

---

## Effect on conversions

Consider:

```cpp
class Base {};
```

### Public inheritance

```cpp
class D : public Base {};

D d;
Base* p = &d; // OK
```

### Protected inheritance

```cpp
class D : protected Base {};

D d;
// Base* p = &d; // Error
```

### Private inheritance

```cpp
class D : private Base {};

D d;
// Base* p = &d; // Error
```

Only public inheritance exposes the base-class interface publicly.

---

## Quick rule

Think of inheritance specifiers as a transformation of the base's interface:

| Inheritance | Base public becomes | Base protected becomes |
| ----------- | ------------------- | ---------------------- |
| public      | public              | protected              |
| protected   | protected           | protected              |
| private     | private             | private                |

And remember:

* **public inheritance** → "is-a" relationship.
* **protected inheritance** → implementation inheritance visible to descendants.
* **private inheritance** → implementation inheritance visible only to the current class.
* **private members of the base are never directly accessible** in the derived class regardless of inheritance type.
