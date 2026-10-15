
# C++ Initialization

C++ initialization is confusing because **`=`, `()`, and `{}` can all initialize an object, but they don't always mean the same thing**.

 The easiest way to understand it is to build a mental model around the major forms.

## 1\. The big picture

 For a type:

```cpp
struct Person {
    std::string name;
    int age;
};
```

 you'll commonly see:

```cpp
Person p5;                      // default initialization
Person p6{};                    // value initialization

Person p3("Narayan", 44);       // direct initialization
Person p4 = Person("Narayan", 44); // copy initialization

Person p1{"Narayan", 44};       // list initialization
Person p2 = {"Narayan", 44};    // copy-list initialization
```

 These look similar, but the rules behind them differ.
 
**Note**
The struct `Person` is called an aggregate in c++ :  since it has 
 * No user-declared or inherited constructors.
 * No private or protected direct non-static data members.
 * No virtual functions or virtual base classes.
---

## 2\. Default initialization

 The simplest form:

```cpp
Person p;
```

 This is **default initialization**.

 What happens depends on the type.

 For your `Person`:

```
struct Person {
    std::string name;
    int age;
};
```

 `p` is default-initialized.

 Since `Person` has no user-provided constructor, its members are themselves default-initialized:

```
p.name; // std::string gets its default constructor
p.age;  // int is NOT initialized
```

 So:

```
Person p;

std::cout << p.age; // ❌ indeterminate value
```

 This is an important distinction.

 For fundamental types:

```
int x;
double d;
bool b;
```

 local variables are **not initialized**.

 But class types get their default constructor if one is available.

 For example:

```
std::string s;
```

 is fine because `std::string` has a default constructor.

---

## 3\. Value initialization

 Now compare:

```
Person p{};
```

 This is **value initialization**.

 The `{}` is significant.

 For your struct, the members that aren't otherwise initialized are value-initialized.

 So:

```
Person p{};

std::cout << p.name; // ""
std::cout << p.age;  // 0
```

 And this is particularly useful for fundamental types:

```
int x{};
double d{};
bool b{};
```

 You get:

```
x == 0
d == 0.0
b == false
```

 This is one of the most useful modern C++ idioms:

```
int count{};
```

 instead of:

```
int count;
```

 because you're explicitly asking for initialization.

---

## 4\. Direct initialization

 You can initialize using parentheses:

```
Person p("Narayan", 44);
```

 This is **direct initialization**.

 For a class, C++ looks for a constructor that can accept those arguments.

Your original struct has **no constructor**:

```
struct Person {
    std::string name;
    int age;
};
```

 So:

```
Person p("Narayan", 44);
```

 will throw an error. 

 But if we add a constructor, For example:

```
struct Person {
    std::string name;
    int age;

    Person(std::string n, int a)
        : name(std::move(n)), age(a) {}
};
```

 Then:

```
Person p("Narayan", 44);
```

 calls:

```
Person::Person(std::string, int)
```

---
## 5\. Copy initialization

```c++
Person p = Person("narayan", 40);
```
First, this is called copy initialization because you use `=` to initialize `p`.  You are initializing `p` using a `prvalue`; However, in modern C++, it does not necessarily mean that an object is copied.

Conceptually, it behaves like this:
```c++
Person p("narayan", 40);
```
Since `Person` struct does not explicitly define a constructor, the expression is invalid.

But if we add a constructor
```c++
struct Person {
    std::string name;
    int age;

    Person(std::string n, int a)
        : name(n), age(a) {}
};
```

```cpp
Person p = Person("narayan", 40);
```
Now the expression on the right creates a `prvalue`,  `Person` object whose members are initialized as follows:
- `name` = `"narayan"`    
- `age` = `40`
    
`p` is initialized from that object 

Conceptually, you might imagine these steps:
1. Construct a temporary `Person` object.    
2. Copy-construct `p` from the temporary object.    
3. Destroy the temporary object.
    
However, modern C++ does not actually perform those steps in this example.

Since C++17, guaranteed copy elision applies here.

The compiler constructs p directly using the specified constructor arguments. There is no separate temporary Person object that gets copied into p.
It acts exactly as
```c++
Person p("narayan", 40);
```

 Here's another example:

```cpp
int x = 42;
```

 This is **copy initialization**.

 Likewise:

```cpp
std::string s = "hello";
```

 The important thing is that initialization is happening at the point where the variable is declared.

 It's not assignment.

 Compare:

```cpp
int x = 42;  // initialization
```

 with:

```cpp
int x;
x = 42;      // assignment
```

 These are fundamentally different operations.

**Note**
```cpp
Person p6 = Person{"narayan", 44};
```
Copy-initialization from a prvalue created by direct-list-initialization

---
## 6\. List initialization

 Now:

```cpp
Person p{"Narayan", 44};
```

 This is **direct-list-initialization**.

 For an aggregate like your `Person`, this performs **aggregate initialization**.

 The members are initialized in order:

```cpp
name <- "Narayan"
age  <- 44
```

 So:

```cpp
struct Person {
    std::string name;
    int age;
};

Person p{"Narayan", 44};
```

 doesn't require a constructor like: 

```cpp
Person(std::string, int);
```

 That's where `{}` is special.  Instead, `Person` is an **aggregate**, and the initializer list initializes its elements.

But if a constructor is present  as in: 

```cpp
struct Person {
    std::string name;
    int age;

    Person(std::string n, int a)
        : name(std::move(n)), age(a) {}
};
```

 Then:

```cpp
Person p{"Narayan", 44};
```

 calls:

```cpp
Person::Person(std::string, int)
```

 This is one of the most important C++ concepts to understand.

---
## 7\. Copy-list initialization

 You can also write:

```cpp
Person p = {"Narayan", 44};
```

 This is **copy-list-initialization**.

 For this example, the result is essentially what you expect:

```cpp
Person p{"Narayan", 44};
```

 The `=` here does **not** mean that a `Person` is first created somewhere else and then copied into `p`.

 That's a common misconception.

 For example:

```cpp
Person p = {"Narayan", 44};
```

 is initialization syntax.

 It's not equivalent to:

```cpp
Person temporary{"Narayan", 44};
Person p = temporary;
```

---

## 8\. Assignment is NOT initialization

 This is worth emphasizing.

```cpp
Person p{"Narayan", 44};
```

 Initialization.

 Later:

```
p = Person{"John", 30};
```

 Assignment.

 The second line uses `Person`'s assignment machinery.

 You can also assign an existing `Person`:

```cpp
Person a{"Narayan", 44};
Person b{"John", 30};

a = b;
```

 Here `a` already exists, so this is **assignment**, not initialization.

---
## 9\. The `new` case

 You'll also encounter:

```cpp
auto* p1 = new Person;
```

 This default-initializes the object.

 For your struct:

```cpp
struct Person {
    std::string name;
    int age;
};
```

 the `std::string` is initialized, but `age` is indeterminate.

 You can instead write:

```cpp
auto* p2 = new Person{};
```

 Now you're value-initializing:

```cpp
p2->name == ""
p2->age == 0
```

 And you can aggregate-initialize:

```cpp
auto* p3 = new Person{"Narayan", 44};
```

---
## 10\. Initialization of individual members

 There's another place where these rules become extremely important:

```cpp
struct Person {
    std::string name;
    int age = 18;
};
```

 The `age = 18` is a **default member initializer**.

 Now:

```cpp
Person p{"Narayan"};
```

 gives:

```cpp
name = "Narayan"
age  = 18
```

 And:

```cpp
Person p{};
```

 gives:

```cpp
name = ""
age  = 18
```

 The default member initializer is used when you don't provide an initializer for that member.

 But:

```cpp
Person p{"Narayan", 44};
```

 overrides it:

```cpp
name = "Narayan"
age  = 44
```

---
## 11\. Why does C++ prefer `{}`?

 Modern C++ generally encourages brace initialization:

```cpp
int x{42};

std::string name{"Narayan"};

Person p{"Narayan", 44};

std::vector<int> numbers{1, 2, 3, 4};
```

 One major reason is that `{}` prevents certain narrowing conversions.

 For example:

```cpp
int x = 3.14;  // allowed, x becomes 3
```

 but:

```cpp
int x{3.14};   // ❌ error
```

 Similarly:

```cpp
char c = 1000;  // potentially allowed with conversion
```

 whereas:

```cpp
char c{1000};   // ❌ narrowing conversion
```

 This is one of the nice safety properties of brace initialization.

---
## 12\. The famous `{}` ambiguity

 There is one C++ gotcha worth knowing.

```cpp
std::vector<int> v(10, 20);
```

 means:

 > Create a vector with **10 elements**, each having value `20`.

 So:

```cpp
[20, 20, 20, 20, ...]  // 10 times
```

 But:

```cpp
std::vector<int> v{10, 20};
```

 means:

 > Create a vector containing the elements `10` and `20`.

 So:

```cpp
[10, 20]
```

#### why?

**`std::initializer_list` constructors generally take precedence over other viable constructors during `list initialization`**

* `std::vector<int> v{10, 20}` calls  `vector(std::initializer_list<T> values)` constructor
* `std::vector<int> v(10, 20)` calls  `vector(size_type count, const T& value)` constructor

This is why `vector<int>{10, 20}` creates two elements instead of ten copies of 20. It's one reason `{}` can sometimes produce surprising results.

----
## 13\. One final distinction: `{}` is not always `initializer_list`

This is important.

Consider:

```cpp
struct Point {
    int x;
    int y;
};

Point p{10, 20};
```

There isn't necessarily an `initializer_list` here.

`Point` can simply be **aggregate-initialized**:

```text
Point
 ├── x = 10
 └── y = 20
```

So `{}` is a general **list-initialization syntax**. `std::initializer_list` is just one of the mechanisms that can participate in it.

That's a subtle but important distinction:

> **List initialization ≠ initializer_list.**

`initializer_list` is a library type:

```cpp
std::initializer_list<T>
```

while **list initialization** is a language feature involving `{}`.

---

### The mental model I'd use

When you see:

```cpp
T x(...);
```

think:

**"Call/select a constructor using parentheses."**

When you see:

```cpp
T x{...};
```

think:

**"Perform list initialization; check initializer-list constructors first."**

And remember:

```cpp
std::vector<int> a(5);   // 5 elements
std::vector<int> b{5};   // 1 element: 5
```

That one example captures a surprisingly large part of why this topic matters.

## 14\. Arrays

 Braces are also the natural syntax for arrays:

```cpp
int numbers[] = {1, 2, 3, 4};
```

 Or:

```cpp
int numbers[4]{1, 2, 3, 4};
or
int numbers[]{1, 2, 3, 4};
```

 You can even partially initialize:

```cpp
int numbers[5]{1, 2};
or
int numbers[]{1, 2};
```

 Result:

```cpp
1 2 0 0 0
```

 The remaining elements are value-initialized.

 And:

```cpp
int numbers[5]{};
```

 gives:

```cpp
0 0 0 0 0
```

---

## 15\. Nested initialization

 This is where the C heritage becomes very obvious.

 Suppose:

```cpp
struct Address {
    std::string city;
    int pin;
};

struct Person {
    std::string name;
    int age;
    Address address;
};
```

 You can write:

```cpp
Person p{
    "Narayan",
    44,
    {"Bengaluru", 560001}
};
```

 The outer braces initialize `Person`, and the inner braces initialize `Address`.

 You can think of it structurally as:

```
Person
├── name    -> "Narayan"
├── age     -> 44
└── address
    ├── city -> "Bengaluru"
    └── pin  -> 560001
```

---

## 16\. A useful mental model

 When you encounter initialization syntax, ask these questions:

 ### `T x;`

```cpp
Person p;
```

 **Default initialization**

 > "Create `p` using the default initialization rules."

---

 ### `T x{};`

```cpp
Person p{};
```

 **Value initialization**

 > "Create `p` and value-initialize it."

---

 ### `T x{a, b};`

```cpp
Person p{"Narayan", 44};
```

 **Direct-list-initialization**

 > "Initialize `p` using `{a, b}`."

 For an aggregate, this becomes **aggregate initialization**.

 For a class with constructors, constructors participate.

---

 ### `T x = {a, b};`

```cpp
Person p = {"Narayan", 44};
```

 **Copy-list-initialization**

 > "Initialize `p` from this braced initializer."

---

 ### `T x(a, b);`

```cpp
Person p("Narayan", 44);
```

 **Direct initialization**

 > "Initialize `p` using these arguments."

 For a class, this normally means constructor overload resolution.

---

 ### `T x = value;`

```cpp
int x = 42;
```

 **Copy initialization**

 > "Initialize `x` from `value`."

---

 ### `x = value;`

```cpp
x = 42;
```

 **Assignment**, not initialization.

---

 # 17\. A cheat sheet

| Syntax          | Name                       | Typical meaning               |
| --------------- | -------------------------- | ----------------------------- |
| `T x;`          | Default initialization     | Default-initialize `x`        |
| `T x{};`        | Value initialization       | Value-initialize `x`          |
| `T x{a, b};`    | Direct-list initialization | Initialize using braces       |
| `T x = {a, b};` | Copy-list initialization   | Initialize using braced list  |
| `T x(a, b);`    | Direct initialization      | Usually invokes a constructor |
| `T x = value;`  | Copy initialization        | Initialize from `value`       |
| `x = value;`    | Assignment                 | Assign to existing object     |

