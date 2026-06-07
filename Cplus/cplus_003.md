# C++ copy and move use cases

In C++, understanding exactly *when* these special member functions are called is the key to mastering resource management (and avoiding accidental performance hits).

Here is the breakdown of the exact scenarios that trigger the copy and move constructors, as well as the copy and move assignment operators.

---

## 1. Copy Constructor

The copy constructor creates a **new object** as a copy of an **existing object** of the same type. It operates on uninitialized memory.

### Triggers:

* **Direct or Copy Initialization:** Creating a new object directly from an existing one.
```cpp
MyClass obj1;
MyClass obj2 = obj1; // Triggers copy constructor
MyClass obj3(obj1);  // Triggers copy constructor

```


* **Passing by Value:** Passing an object into a function by value (not by reference).
```cpp
void printObject(MyClass obj) { /* ... */ }

MyClass mainObj;
printObject(mainObj); // Triggers copy constructor to create 'obj'

```


* **Returning by Value (Pre-C++17 / No Elision):** Returning an object from a function by value. *Note: Modern compilers often optimize this away via Copy Elision/NRVO, but conceptually, it relies on the copy constructor.*
```cpp
MyClass createObject() {
    MyClass localObj;
    return localObj; // Can trigger copy constructor (if not elided)
}

```


* **Catching Exceptions by Value:** ```cpp
try {
throw MyClass();
} catch (MyClass ex) { // Triggers copy constructor
// ...
}
```


```



---

## 2. Copy Assignment Operator (`operator=`)

Unlike the constructor, the copy assignment operator copies values from an existing object into an **already initialized, existing object**.

### Triggers:

* **Assigning one existing variable to another:**
```cpp
MyClass obj1;
MyClass obj2; 
// Both objects already exist
obj2 = obj1; // Triggers copy assignment operator

```



> ⚠️ **Common Gotcha:** `MyClass obj2 = obj1;` looks like an assignment because of the `=` sign, but because a new object (`obj2`) is being created, it calls the **copy constructor**, not the assignment operator.

---

## 3. Move Constructor

Introduced in C++11, the move constructor creates a **new object** by "stealing" resources from a temporary or explicitly moved object (**rvalue**), leaving the source in a valid but unspecified state.

### Triggers:

* **Initialization with a Temporary (Rvalue):**
```cpp
MyClass obj = MyClass(); // Triggers move constructor (if not elided by compiler)

```


* **Using `std::move`:** Explicitly casting an lvalue to an rvalue reference to force a move.
```cpp
MyClass obj1;
MyClass obj2 = std::move(obj1); // Triggers move constructor

```


* **Returning a Local Object by Value:** If the compiler cannot perform Copy Elision, it will automatically attempt to use the move constructor to return the local variable.
```cpp
MyClass makeObject() {
    MyClass localObj;
    return localObj; // Automatically moved if not elided
}

```



---

## 4. Move Assignment Operator (`operator=`)

The move assignment operator transfers resources from a temporary or explicitly moved object (**rvalue**) into an **already initialized, existing object**.

### Triggers:

* **Assigning a Temporary to an Existing Object:**
```cpp
MyClass obj;
obj = MyClass(); // Triggers move assignment (temporary is destroyed right after)

```


* **Assigning via `std::move` to an Existing Object:**
```cpp
MyClass obj1;
MyClass obj2;
// Both objects already exist
obj2 = std::move(obj1); // Triggers move assignment

```



---

## Quick Reference Summary

| Special Member Function | Target Object Status | Source Object Type       | Example Syntax              |
| ----------------------- | -------------------- | ------------------------ | --------------------------- |
| **Copy Constructor**    | Being Created (New)  | Lvalue (Existing/Named)  | `MyClass a = b;`            |
| **Move Constructor**    | Being Created (New)  | Rvalue (Temporary/Moved) | `MyClass a = std::move(b);` |
| **Copy Assignment**     | Already Initialized  | Lvalue (Existing/Named)  | `a = b;`                    |
| **Move Assignment**     | Already Initialized  | Rvalue (Temporary/Moved) | `a = std::move(b);`         |