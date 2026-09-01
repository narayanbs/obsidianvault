 The easiest way to learn Go reflection is to build a mental model around just **four things**:

1. `reflect.Type`
2. `reflect.Value`
3. `Kind`
4. `Interface`

Here’s a practical tutorial from beginner → useful real-world patterns.

---

# Go Reflection Tutorial

Go reflection is provided by the standard [`reflect`](https://pkg.go.dev/reflect) package.

You typically start with:

```go
import "reflect"
```

## 1. What is reflection?

Normally, Go is strongly typed:

```go
func printInt(x int) {
    fmt.Println(x)
}
```

You can't pass a `string` to it.

Reflection lets you inspect a value **at runtime**, even when you don't know its concrete type at compile time.

For example:

```go
func inspect(x interface{}) {
    fmt.Println(reflect.TypeOf(x))
    fmt.Println(reflect.ValueOf(x))
}
```

Then:

```go
inspect(42)
inspect("hello")
inspect(true)
```

Output:

```text
int
42

string
hello

bool
true
```

Think of reflection as:

> **"I don't know what this is at compile time; let me find out at runtime."**

---

# 2. `reflect.Type`

`reflect.Type` describes a type.

```go
x := 42

t := reflect.TypeOf(x)

fmt.Println(t)
fmt.Println(t.Name())
fmt.Println(t.Kind())
```

Output:

```text
int
int
int
```

Some useful methods:

```go
t.Name()
t.Kind()
t.String()
t.Size()
t.NumMethod()
```

For structs:

```go
t.NumField()
t.Field(0)
t.FieldByName("Name")
```

---

# 3. `reflect.Value`

`reflect.Value` represents an actual value.

```go
x := 42

v := reflect.ValueOf(x)

fmt.Println(v)
fmt.Println(v.Int())
fmt.Println(v.Kind())
```

Output:

```text
42
42
int
```

You can think of it this way:

```text
             42
              |
       +------+------+
       |             |
   reflect.Type   reflect.Value
       |             |
      int            42
```

`Type` tells you **what it is**.

`Value` gives you access to **the value itself**.

---

# 4. `Kind`

This is extremely important.

```go
reflect.TypeOf(42).Kind()
```

returns:

```go
reflect.Int
```

Likewise:

```go
reflect.TypeOf("hello").Kind()
```

returns:

```go
reflect.String
```

And:

```go
reflect.TypeOf([]int{}).Kind()
```

returns:

```go
reflect.Slice
```

`Kind()` tells you the **general category** of a type.

Common kinds:

```go
reflect.Bool

reflect.Int
reflect.Int8
reflect.Int16
reflect.Int32
reflect.Int64

reflect.Uint
reflect.Uint8
reflect.Uint16
reflect.Uint32
reflect.Uint64

reflect.Float32
reflect.Float64

reflect.String

reflect.Array
reflect.Slice
reflect.Map
reflect.Struct
reflect.Pointer
reflect.Interface
reflect.Func
reflect.Chan

reflect.Invalid
```

---

# 5. Type vs Kind

This distinction causes a lot of confusion.

Consider:

```go
type UserID int
```

Now:

```go
var id UserID = 10

t := reflect.TypeOf(id)

fmt.Println(t.Name())
fmt.Println(t.Kind())
```

Output:

```text
UserID
int
```

So:

```text
Type = UserID
Kind = int
```

**Type** is specific.

**Kind** is the underlying category.

This is why reflection code often does:

```go
switch v.Kind() {
case reflect.Int:
    // ...
case reflect.String:
    // ...
case reflect.Struct:
    // ...
}
```

---

# 6. `TypeOf` vs `ValueOf`

You'll use these constantly.

```go
x := 100

t := reflect.TypeOf(x)
v := reflect.ValueOf(x)
```

Think:

```text
reflect.TypeOf(x)
        ↓
     "int"

reflect.ValueOf(x)
        ↓
      "100"
```

So if you only want to know the type:

```go
t := reflect.TypeOf(x)
```

If you need to manipulate/read the value:

```go
v := reflect.ValueOf(x)
```

---

# 7. Your original question

You asked about:

```go
reflect.TypeOf(o).Kind()
```

vs

```go
reflect.ValueOf(0).Kind()
```

Usually:

```go
v := reflect.ValueOf(o)

switch v.Kind() {
case reflect.String:
    ...
case reflect.Int:
    ...
}
```

is convenient if you're going to work with the value afterward.

For example:

```go
func inspect(x interface{}) {
    v := reflect.ValueOf(x)

    switch v.Kind() {
    case reflect.Int:
        fmt.Println("integer:", v.Int())

    case reflect.String:
        fmt.Println("string:", v.String())

    case reflect.Bool:
        fmt.Println("boolean:", v.Bool())
    }
}
```

---

# 8. Reading values

`reflect.Value` has methods for extracting values.

### Integers

```go
v.Int()
```

### Unsigned integers

```go
v.Uint()
```

### Floats

```go
v.Float()
```

### Strings

```go
v.String()
```

### Booleans

```go
v.Bool()
```

Example:

```go
func printValue(x interface{}) {
    v := reflect.ValueOf(x)

    switch v.Kind() {
    case reflect.Int:
        fmt.Println(v.Int())

    case reflect.String:
        fmt.Println(v.String())

    case reflect.Bool:
        fmt.Println(v.Bool())
    }
}
```

---

# 9. `Interface()`

Sometimes you want to convert a `reflect.Value` back to a normal Go value.

```go
x := 42

v := reflect.ValueOf(x)

y := v.Interface()

fmt.Printf("%T\n", y)
```

Output:

```text
int
```

You can think:

```text
normal Go value
      ↓
reflect.ValueOf()
      ↓
reflect.Value
      ↓
Interface()
      ↓
normal Go value
```

For example:

```go
func print(x interface{}) {
    v := reflect.ValueOf(x)

    actual := v.Interface()

    fmt.Printf("%T = %v\n", actual, actual)
}
```

---

# 10. Reflection and structs

This is where reflection becomes really useful.

Suppose:

```go
type User struct {
    Name string
    Age  int
}
```

Then:

```go
user := User{
    Name: "John",
    Age:  30,
}

v := reflect.ValueOf(user)
```

You can inspect fields:

```go
fmt.Println(v.NumField())
```

Output:

```text
2
```

Get a field:

```go
name := v.Field(0)

fmt.Println(name)
fmt.Println(name.String())
```

Output:

```text
John
John
```

Or:

```go
age := v.Field(1)

fmt.Println(age.Int())
```

---

# 11. Get struct fields by name

This is usually more readable:

```go
name := v.FieldByName("Name")

fmt.Println(name.String())
```

And:

```go
age := v.FieldByName("Age")

fmt.Println(age.Int())
```

A common reflection pattern:

```go
func printStruct(x interface{}) {
    v := reflect.ValueOf(x)
    t := reflect.TypeOf(x)

    for i := 0; i < v.NumField(); i++ {
        field := t.Field(i)
        value := v.Field(i)

        fmt.Println(field.Name, value.Interface())
    }
}
```

Usage:

```go
printStruct(User{
    Name: "John",
    Age: 30,
})
```

Output:

```text
Name John
Age 30
```

---

# 12. Struct tags

Reflection is heavily used for struct tags.

For example:

```go
type User struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```

You can inspect the tags:

```go
t := reflect.TypeOf(User{})

field, _ := t.FieldByName("Name")

fmt.Println(field.Tag.Get("json"))
```

Output:

```text
name
```

This is one of the main reasons reflection exists in Go.

Libraries such as JSON serializers, ORMs, validators, etc. commonly use reflection and struct tags.

---

# 13. Slices

Suppose:

```go
numbers := []int{10, 20, 30}
```

Then:

```go
v := reflect.ValueOf(numbers)
```

Check:

```go
fmt.Println(v.Kind())
```

Output:

```text
slice
```

Length:

```go
fmt.Println(v.Len())
```

Access elements:

```go
fmt.Println(v.Index(0).Int())
fmt.Println(v.Index(1).Int())
fmt.Println(v.Index(2).Int())
```

You can loop:

```go
for i := 0; i < v.Len(); i++ {
    fmt.Println(v.Index(i).Interface())
}
```

---

# 14. Maps

Suppose:

```go
m := map[string]int{
    "Alice": 10,
    "Bob":   20,
}
```

Reflection:

```go
v := reflect.ValueOf(m)

fmt.Println(v.Kind())
```

Output:

```text
map
```

Get a value:

```go
key := reflect.ValueOf("Alice")

value := v.MapIndex(key)

fmt.Println(value.Int())
```

Output:

```text
10
```

Iterate:

```go
for _, key := range v.MapKeys() {
    value := v.MapIndex(key)

    fmt.Println(key.Interface(), value.Interface())
}
```

---

# 15. Pointers

Pointers are very important with reflection.

Consider:

```go
x := 10

v := reflect.ValueOf(&x)

fmt.Println(v.Kind())
```

Output:

```text
ptr
```

Get the value pointed to:

```go
elem := v.Elem()

fmt.Println(elem.Kind())
fmt.Println(elem.Int())
```

Output:

```text
int
10
```

So:

```text
&x
 ↓
reflect.Value
 ↓
Elem()
 ↓
x
```

---

# 16. The biggest reflection trap: `CanSet`

Consider:

```go
x := 10

v := reflect.ValueOf(x)

v.SetInt(20)
```

This will panic.

Why?

Because `reflect.ValueOf(x)` contains a copy of `x`, and that value isn't settable.

To modify `x`, you need a pointer:

```go
x := 10

v := reflect.ValueOf(&x)

v = v.Elem()

v.SetInt(20)

fmt.Println(x)
```

Output:

```text
20
```

The important pattern is:

```go
v := reflect.ValueOf(&x)
v = v.Elem()

v.SetInt(20)
```

---

# 17. `CanSet()`

Before modifying a value, you can check:

```go
if v.CanSet() {
    v.SetInt(20)
}
```

Example:

```go
func setInt(x interface{}) {
    v := reflect.ValueOf(x)

    if v.Kind() != reflect.Ptr {
        return
    }

    v = v.Elem()

    if v.CanSet() && v.Kind() == reflect.Int {
        v.SetInt(100)
    }
}
```

Usage:

```go
x := 10

setInt(&x)

fmt.Println(x)
```

Output:

```text
100
```

---

# 18. `IsValid()`

Remember this:

```go
reflect.ValueOf(nil)
```

produces an invalid `reflect.Value`.

Therefore:

```go
v := reflect.ValueOf(nil)

fmt.Println(v.IsValid())
```

returns:

```text
false
```

Calling some methods on an invalid value can panic.

A safe reflection function often starts with:

```go
v := reflect.ValueOf(x)

if !v.IsValid() {
    return
}
```

---

# 19. Nil pointers/interfaces

Another important issue:

```go
var p *int = nil

v := reflect.ValueOf(p)

fmt.Println(v.Kind()) // ptr
fmt.Println(v.IsNil()) // true
```

Notice:

```go
v.Kind() == reflect.Pointer
```

but the pointer itself is nil.

For kinds that can be nil, you can use:

```go
v.IsNil()
```

This applies to things like:

```text
Pointer
Map
Slice
Interface
Func
Chan
```

---

# 20. Interfaces are tricky

Consider:

```go
var x interface{} = 42

v := reflect.ValueOf(x)

fmt.Println(v.Kind())
```

You might expect:

```text
interface
```

But you get:

```text
int
```

Why?

Because `reflect.ValueOf()` gives you the **dynamic value stored inside the interface**.

Conceptually:

```text
interface{}
   |
   +---- dynamic value: int(42)
                         ↓
                  reflect.ValueOf()
                         ↓
                        int
```

This is one of the most important things to understand about Go reflection.

---

# 21. A practical reflection function

Here's a useful example combining the concepts:

```go
func inspect(x interface{}) {
    v := reflect.ValueOf(x)

    if !v.IsValid() {
        fmt.Println("nil")
        return
    }

    fmt.Println("Type:", v.Type())
    fmt.Println("Kind:", v.Kind())

    switch v.Kind() {
    case reflect.Int:
        fmt.Println("Value:", v.Int())

    case reflect.String:
        fmt.Println("Value:", v.String())

    case reflect.Bool:
        fmt.Println("Value:", v.Bool())

    case reflect.Slice:
        fmt.Println("Length:", v.Len())

    case reflect.Map:
        fmt.Println("Length:", v.Len())

    case reflect.Struct:
        fmt.Println("Fields:", v.NumField())
    }
}
```

You could call:

```go
inspect(42)
inspect("hello")
inspect(true)
inspect([]int{1, 2, 3})
inspect(map[string]int{"a": 1})
inspect(User{})
```

---

# 22. Reflection decision tree

When you're writing reflection code, this mental flow is very useful:

```text
                 reflect.ValueOf(x)
                         |
                         v
                  IsValid() ?
                   /       \
                 no         yes
                 |           |
                nil       Kind()
                             |
          +------------------+------------------+
          |         |         |        |         |
         Ptr      Struct     Slice    Map      Basic
          |         |         |        |         |
        Elem()   Fields      Index    Keys     Int/String/...
```

And for types:

```text
reflect.TypeOf(x)
       |
       +-- Kind()
       |
       +-- Name()
       |
       +-- NumField()
       |
       +-- Field()
       |
       +-- NumMethod()
       |
       +-- Method()
```

---

# 23. When should you use reflection?

Reflection is useful when you need **generic runtime behavior**.

Common examples:

### JSON libraries

```go
type User struct {
    Name string `json:"name"`
}
```

A serializer can inspect fields and tags dynamically.

### Validation

```go
type User struct {
    Name string `validate:"required"`
}
```

A validation library can inspect the `validate` tag.

### ORMs

An ORM can inspect:

```go
type User struct {
    ID   int
    Name string
}
```

and dynamically map fields to database columns.

### Generic utilities

For example:

```go
func contains(slice interface{}, target interface{}) bool
```

Reflection can inspect an arbitrary slice.

---

# 24. When NOT to use reflection

This is equally important.

Don't use reflection just because it's possible.

For example, don't do this:

```go
func add(a, b interface{}) interface{} {
    // reflection...
}
```

if you can simply use:

```go
func add(a, b int) int {
    return a + b
}
```

Reflection makes code:

* harder to understand
* harder to maintain
* less type-safe
* often slower
* easier to make panic

Prefer **generics** when the problem can be solved with compile-time type parameters.

---

# 25. Reflection vs Generics

Modern Go has generics, so this distinction matters.

Instead of reflection:

```go
func first(x interface{}) interface{} {
    v := reflect.ValueOf(x)
    return v.Index(0).Interface()
}
```

you might be able to use:

```go
func first[T any](x []T) T {
    return x[0]
}
```

Generics should generally be your first choice when the behavior can be expressed at compile time.

Reflection is appropriate when the type is genuinely unknown until runtime.

---

# 26. The 10 methods I recommend learning first

Don't try to memorize the entire `reflect` package.

Start with these:

### `reflect.Type`

```go
reflect.TypeOf(x)
```

### `reflect.Value`

```go
reflect.ValueOf(x)
```

### `Kind`

```go
v.Kind()
```

### `Type`

```go
v.Type()
```

### `Interface`

```go
v.Interface()
```

### `Elem`

```go
v.Elem()
```

### `Field`

```go
v.Field(i)
```

### `FieldByName`

```go
v.FieldByName("Name")
```

### `Len`

```go
v.Len()
```

### `CanSet`

```go
v.CanSet()
```

Then learn:

```go
IsValid()
IsNil()
Set()
SetInt()
SetString()
Index()
MapIndex()
NumField()
NumMethod()
```

---

# 27. One rule that will save you from many reflection panics

Before doing reflection operations, **check the kind**.

Don't blindly do:

```go
v.Int()
```

because this panics if `v` isn't an integer.

Instead:

```go
if v.Kind() == reflect.Int {
    fmt.Println(v.Int())
}
```

Similarly:

```go
if v.Kind() == reflect.Struct {
    fmt.Println(v.NumField())
}
```

And:

```go
if v.Kind() == reflect.Slice {
    fmt.Println(v.Len())
}
```

---

# 28. A small exercise

Try implementing this:

```go
func describe(x interface{}) {
    // Print:
    //
    // Type: ...
    // Kind: ...
    //
    // If struct:
    // Fields: ...
    //
    // If slice:
    // Length: ...
}
```

For:

```go
type Person struct {
    Name string
    Age  int
}

describe(Person{
    Name: "Alice",
    Age:  25,
})

describe([]int{10, 20, 30})

describe("hello")

describe(42)
```

Expected idea:

```text
Type: main.Person
Kind: struct
Fields: 2

Type: []int
Kind: slice
Length: 3

Type: string
Kind: string

Type: int
Kind: int
```

If you understand how to implement that, you're already getting comfortable with the core of Go reflection.

### The mental model to remember

```text
reflect.Type
    ↓
"What type is this?"

reflect.Value
    ↓
"What value do I have?"

Kind
    ↓
"What category of type is it?"

Elem
    ↓
"What is inside this pointer/interface?"

Interface
    ↓
"Give me the normal Go value"

CanSet
    ↓
"Can I modify it?"
```

**One especially useful rule:** if you're writing reflection-heavy Go code, start by getting a `reflect.Value`, check `IsValid()`/`Kind()`, and then operate on it. That pattern will make most reflection code much easier to reason about.
