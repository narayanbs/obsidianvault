C++ initialization is confusing because **`=`, `()`, and `{}` can all initialize an object, but they don't always mean the same thing**.

 The easiest way to understand it is to build a mental model around the major forms.

## 1\. The big picture

 For a type:

```
struct Person {
    std::string name;
    int age;
};
```

 you'll commonly see:

```
Person p1{"Narayan", 44};       // list initialization
Person p2 = {"Narayan", 44};    // copy-list initialization

Person p3("Narayan", 44);       // direct initialization
Person p4 = Person("Narayan", 44); // copy initialization

Person p5;                      // default initialization
Person p6{};                    // value initialization
```

 These look similar, but the rules behind them differ.

---

## 2\. Default initialization

 The simplest form:

```
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

 # 3\. Value initialization

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

 # 4\. Direct initialization

 You can initialize using parentheses:

```
Person p("Narayan", 44);
```

 This is **direct initialization**.

 For a class, C++ looks for a constructor that can accept those arguments.

 For example:

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

 ### Important connection to your original question

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

 doesn't mean "initialize the fields."

 That's where `{}` is special.

---

 # 5\. List initialization

 Now:

```
Person p{"Narayan", 44};
```

 This is **direct-list-initialization**.

 For an aggregate like your `Person`, this performs **aggregate initialization**.

 The members are initialized in order:

```
name <- "Narayan"
age  <- 44
```

 So:

```
struct Person {
    std::string name;
    int age;
};

Person p{"Narayan", 44};
```

 doesn't require a constructor like:

```
Person(std::string, int);
```

 Instead, `Person` is an **aggregate**, and the initializer list initializes its elements.

 This is one of the most important C++ concepts to understand.

---

 # 6\. Copy-list initialization

 You can also write:

```
Person p = {"Narayan", 44};
```

 This is **copy-list-initialization**.

 For this example, the result is essentially what you expect:

```
Person p{"Narayan", 44};
```

 The `=` here does **not** mean that a `Person` is first created somewhere else and then copied into `p`.

 That's a common misconception.

 For example:

```
Person p = {"Narayan", 44};
```

 is initialization syntax.

 It's not equivalent to:

```
Person temporary{"Narayan", 44};
Person p = temporary;
```

---

 # 7\. Copy initialization

 Here's another form:

```
int x = 42;
```

 This is **copy initialization**.

 Likewise:

```
std::string s = "hello";
```

 The important thing is that initialization is happening at the point where the variable is declared.

 It's not assignment.

 Compare:

```
int x = 42;  // initialization
```

 with:

```
int x;
x = 42;      // assignment
```

 These are fundamentally different operations.

 For class types this distinction becomes much more interesting.

---

 # 8\. Assignment is NOT initialization

 This is worth emphasizing.

```
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

```
Person a{"Narayan", 44};
Person b{"John", 30};

a = b;
```

 Here `a` already exists, so this is **assignment**, not initialization.

---

 # 9\. The `new` case

 You'll also encounter:

```
auto* p1 = new Person;
```

 This default-initializes the object.

 For your struct:

```
struct Person {
    std::string name;
    int age;
};
```

 the `std::string` is initialized, but `age` is indeterminate.

 You can instead write:

```
auto* p2 = new Person{};
```

 Now you're value-initializing:

```
p2->name == ""
p2->age == 0
```

 And you can aggregate-initialize:

```
auto* p3 = new Person{"Narayan", 44};
```

---

 # 10\. Initialization of individual members

 There's another place where these rules become extremely important:

```
struct Person {
    std::string name;
    int age = 18;
};
```

 The `age = 18` is a **default member initializer**.

 Now:

```
Person p{"Narayan"};
```

 gives:

```
name = "Narayan"
age  = 18
```

 And:

```
Person p{};
```

 gives:

```
name = ""
age  = 18
```

 The default member initializer is used when you don't provide an initializer for that member.

 But:

```
Person p{"Narayan", 44};
```

 overrides it:

```
name = "Narayan"
age  = 44
```

---

 # 11\. Constructor initialization

 Once you introduce a constructor:

```
struct Person {
    std::string name;
    int age;

    Person(std::string n, int a)
        : name(std::move(n)),
          age(a)
    {}
};
```

 then:

```
Person p{"Narayan", 44};
```

 will call the constructor.

 This is an important point:

 ### Same syntax, potentially different mechanism

```
Person p{"Narayan", 44};
```

 If `Person` is an aggregate:

 > aggregate initialization

 If `Person` has an appropriate constructor:

 > constructor initialization

 The `{}` syntax itself doesn't tell you which mechanism is ultimately being used.

---

 # 12\. Why does C++ prefer `{}`?

 Modern C++ generally encourages brace initialization:

```
int x{42};

std::string name{"Narayan"};

Person p{"Narayan", 44};

std::vector<int> numbers{1, 2, 3, 4};
```

 One major reason is that `{}` prevents certain narrowing conversions.

 For example:

```
int x = 3.14;  // allowed, x becomes 3
```

 but:

```
int x{3.14};   // ❌ error
```

 Similarly:

```
char c = 1000;  // potentially allowed with conversion
```

 whereas:

```
char c{1000};   // ❌ narrowing conversion
```

 This is one of the nice safety properties of brace initialization.

---

 # 13\. The famous `{}` ambiguity

 There is one C++ gotcha worth knowing.

```
std::vector<int> v(10, 20);
```

 means:

 > Create a vector with **10 elements**, each having value `20`.

 So:

```
[20, 20, 20, 20, ...]  // 10 times
```

 But:

```
std::vector<int> v{10, 20};
```

 means:

 > Create a vector containing the elements `10` and `20`.

 So:

```
[10, 20]
```

 This happens because constructors and `std::initializer_list` interact with brace initialization.

 It's one reason `{}` can sometimes produce surprising results.

---

 # 14\. Arrays

 Braces are also the natural syntax for arrays:

```
int numbers[] = {1, 2, 3, 4};
```

 Or:

```
int numbers[4]{1, 2, 3, 4};
```

 You can even partially initialize:

```
int numbers[5]{1, 2};
```

 Result:

```
1 2 0 0 0
```

 The remaining elements are value-initialized.

 And:

```
int numbers[5]{};
```

 gives:

```
0 0 0 0 0
```

---

 # 15\. Nested initialization

 This is where the C heritage becomes very obvious.

 Suppose:

```
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

```
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

 # 16\. A useful mental model

 When you encounter initialization syntax, ask these questions:

 ### `T x;`

```
Person p;
```

 **Default initialization**

 > "Create `p` using the default initialization rules."

---

 ### `T x{};`

```
Person p{};
```

 **Value initialization**

 > "Create `p` and value-initialize it."

---

 ### `T x{a, b};`

```
Person p{"Narayan", 44};
```

 **Direct-list-initialization**

 > "Initialize `p` using `{a, b}`."

 For an aggregate, this becomes **aggregate initialization**.

 For a class with constructors, constructors participate.

---

 ### `T x = {a, b};`

```
Person p = {"Narayan", 44};
```

 **Copy-list-initialization**

 > "Initialize `p` from this braced initializer."

---

 ### `T x(a, b);`

```
Person p("Narayan", 44);
```

 **Direct initialization**

 > "Initialize `p` using these arguments."

 For a class, this normally means constructor overload resolution.

---

 ### `T x = value;`

```
int x = 42;
```

 **Copy initialization**

 > "Initialize `x` from `value`."

---

 ### `x = value;`

```
x = 42;
```

 **Assignment**, not initialization.

---

 # 17\. A cheat sheet

| Syntax | Name | Typical meaning |
| --- | --- | --- |
| `T x;` | Default initialization | Default-initialize `x` |
| `T x{};` | Value initialization | Value-initialize `x` |
| `T x{a, b};` | Direct-list initialization | Initialize using braces |
| `T x = {a, b};` | Copy-list initialization | Initialize using braced list |
| `T x(a, b);` | Direct initialization | Usually invokes a constructor |
| `T x = value;` | Copy initialization | Initialize from `value` |
| `x = value;` | Assignment | Assign to existing object |

And for your original example:

```
struct Person {
    std::string name;
    int age;
};

Person p{"Narayan", 44};
```

 the key chain is:

```
Person p{...}
       ↓
direct-list-initialization
       ↓
Person is an aggregate
       ↓
aggregate initialization
       ↓
name ← "Narayan"
age  ← 44
```

 **No automatically generated `Person(std::string, int)` constructor is needed.**

 One final distinction that's particularly useful when learning C++: **"initialization syntax" and "the mechanism that performs initialization" are separate concepts.** `{}` is syntax; depending on the type and context, it can lead to aggregate initialization, constructor invocation, value initialization, `initializer_list` construction, etc. That's why C++ initialization initially feels much more complicated than C's `struct x = {...}`.