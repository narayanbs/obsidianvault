
# Go Generics - Complete Tutorial

> [!info]
> **Prerequisites**
> - Go 1.18+
> - Basic understanding of functions, structs, interfaces, and slices

---

# Table of Contents

- Why Generics?
- Type Parameters
- Generic Functions
- Generic Types
- Constraints
- Built-in Constraints
- Custom Constraints
- Union Types
- Approximation (`~`)
- Type Sets
- Type Inference
- Methods on Generic Types
- Generic Interfaces
- Generic Data Structures
- Practical Examples
- Common Patterns
- Limitations
- Best Practices
- Performance
- Generics vs Interfaces

---

# Why Generics?

Before Go 1.18, reusable code often required duplication.

Instead of writing:

```go
func SumInts(nums []int) int {
    var s int
    for _, v := range nums {
        s += v
    }
    return s
}

func SumFloat64(nums []float64) float64 {
    var s float64
    for _, v := range nums {
        s += v
    }
    return s
}
```

You can write:

```go
func Sum[T int | float64](nums []T) T {
    var s T
    for _, v := range nums {
        s += v
    }
    return s
}
```

Usage:

```go
fmt.Println(Sum([]int{1,2,3}))
fmt.Println(Sum([]float64{1.1,2.2}))
```

---

# Type Parameters

Type parameters appear inside square brackets.

```go
func Print[T any](value T) {
    fmt.Println(value)
}
```

`T` is the type parameter.

`any` means every type is allowed.

It is simply an alias for:

```go
interface{}
```

## Multiple Type Parameters

```go
func Pair[A any, B any](a A, b B) {
    fmt.Println(a, b)
}
```

Usage:

```go
Pair(10, "hello")
Pair(true, 4.5)
```

---

# Generic Functions

Example:

```go
func Identity[T any](v T) T {
    return v
}
```

Usage:

```go
x := Identity(10)
y := Identity("Go")
```

The compiler infers:

```
T = int
T = string
```

Explicit type arguments also work:

```go
Identity[int](10)
```

---

# Generic Types

## Generic Struct

```go
type Box[T any] struct {
    Value T
}
```

Usage:

```go
i := Box[int]{Value: 10}
s := Box[string]{Value: "hello"}
```

---

Another example:

```go
type Pair[T any] struct {
    First  T
    Second T
}
```

Works for:

```go
Pair[int]
Pair[string]
Pair[Employee]
```

---

# Constraints

This does **not** compile:

```go
func Add[T any](a, b T) T {
    return a + b
}
```

Because `any` doesn't guarantee the `+` operator exists.

Correct version:

```go
func Add[T int | float64](a, b T) T {
    return a + b
}
```

---

# Built-in Constraints

## `any`

Allows every type.

```go
func Print[T any](v T)
```

---

## `comparable`

Allows equality operations.

```go
func Equal[T comparable](a, b T) bool {
    return a == b
}
```

### Works for

- int
- string
- bool
- pointers
- arrays
- comparable structs

### Doesn't work for

- slices
- maps
- functions

Example:

```go
Equal([]int{1}, []int{1}) // Compile error
```

---

# Custom Constraints

```go
type Number interface {
    int | int64 | float32 | float64
}
```

Now:

```go
func Sum[T Number](a, b T) T {
    return a + b
}
```

---

# Union Types

Constraints can define multiple allowed types.

```go
type Integer interface {
    int |
    int8 |
    int16 |
    int32 |
    int64
}
```

Or combine constraints:

```go
type Numeric interface {
    Integer | Float
}
```

---

# Approximation (`~`)

This is one of the most important generic features.

```go
type MyInt int
```

Without approximation:

```go
type Number interface {
    int
}
```

Only accepts:

```
int
```

Not:

```
MyInt
```

---

With approximation:

```go
type Number interface {
    ~int
}
```

Now all types whose underlying type is `int` are accepted.

Example:

```go
type Age int

func Double[T ~int](v T) T {
    return v * 2
}

var age Age = 20

fmt.Println(Double(age))
```

---

# Type Sets

Constraints define a set of allowed types.

```go
type Signed interface {
    ~int |
    ~int8 |
    ~int16 |
    ~int32 |
    ~int64
}
```

Allowed:

- int
- MyInt
- Age
- Salary

Anything with an underlying signed integer type.

---

# Type Inference

Usually you don't specify type arguments.

```go
Max(3, 4)
```

Compiler infers:

```
T = int
```

Explicit version:

```go
Max[int](3,4)
```

---

# Methods on Generic Types

```go
type Stack[T any] struct {
    items []T
}
```

Push:

```go
func (s *Stack[T]) Push(v T) {
    s.items = append(s.items, v)
}
```

Pop:

```go
func (s *Stack[T]) Pop() T {
    n := len(s.items)
    value := s.items[n-1]
    s.items = s.items[:n-1]
    return value
}
```

Usage:

```go
var s Stack[int]

s.Push(10)
s.Push(20)

fmt.Println(s.Pop())
```

---

# Generic Interfaces

```go
type Reader[T any] interface {
    Read() T
}
```

Implementation:

```go
type IntReader struct{}

func (IntReader) Read() int {
    return 100
}
```

---

# Generic Linked List

```go
type Node[T any] struct {
    Value T
    Next  *Node[T]
}
```

Usage:

```go
head := &Node[int]{Value: 1}

head.Next = &Node[int]{
    Value: 2,
}
```

---

# Generic Queue

```go
type Queue[T any] struct {
    data []T
}

func (q *Queue[T]) Enqueue(v T) {
    q.data = append(q.data, v)
}

func (q *Queue[T]) Dequeue() T {
    value := q.data[0]
    q.data = q.data[1:]
    return value
}
```

---

# Generic Map Function

```go
func Map[T any, U any](items []T, f func(T) U) []U {

    result := make([]U, len(items))

    for i, v := range items {
        result[i] = f(v)
    }

    return result
}
```

Example:

```go
nums := []int{1,2,3}

strings := Map(nums, func(v int) string {
    return strconv.Itoa(v)
})
```

---

# Generic Filter

```go
func Filter[T any](items []T, keep func(T) bool) []T {

    result := []T{}

    for _, v := range items {
        if keep(v) {
            result = append(result, v)
        }
    }

    return result
}
```

---

# Generic Reduce

```go
func Reduce[T any](items []T, init T, f func(T, T) T) T {

    acc := init

    for _, v := range items {
        acc = f(acc, v)
    }

    return acc
}
```

---

# Ordered Constraint

Go 1.21 introduced `cmp.Ordered`.

```go
import "cmp"

func Max[T cmp.Ordered](a, b T) T {
    if a > b {
        return a
    }
    return b
}
```

Supports:

- integers
- floats
- strings

---

# Generic Binary Search Tree

```go
import "cmp"

type Node[T cmp.Ordered] struct {
    Value T
    Left  *Node[T]
    Right *Node[T]
}

func Insert[T cmp.Ordered](n *Node[T], v T) *Node[T] {

    if n == nil {
        return &Node[T]{Value: v}
    }

    if v < n.Value {
        n.Left = Insert(n.Left, v)
    } else {
        n.Right = Insert(n.Right, v)
    }

    return n
}
```

---

# Generic Set

```go
type Set[T comparable] map[T]struct{}

func (s Set[T]) Add(v T) {
    s[v] = struct{}{}
}

func (s Set[T]) Contains(v T) bool {
    _, ok := s[v]
    return ok
}
```

Usage:

```go
s := Set[string]{}

s.Add("Go")

fmt.Println(s.Contains("Go"))
```

---

# Common Patterns

## Result

```go
type Result[T any] struct {
    Value T
    Err   error
}
```

---

## Optional

```go
type Optional[T any] struct {
    Value T
    Valid bool
}
```

---

## Cache

```go
type Cache[K comparable, V any] map[K]V
```

---

## Repository

```go
type Repository[T any] interface {
    Save(T) error
    Find(int) (T, error)
}
```

---

# Limitations

> [!warning]

- No specialization
- No generic methods on non-generic types
- No covariance or contravariance
- No operator overloading
- No runtime type parameter inspection

---

# Best Practices

> [!tip]

- Use generics to eliminate duplication.
- Prefer standard constraints.
- Keep constraints minimal.
- Let the compiler infer type arguments.
- Use interfaces when modeling behavior.

---

# Performance

Generics provide:

- Compile-time type safety
- No repeated type assertions
- Fewer interface conversions
- Optimized machine code

Benchmark performance-critical code instead of assuming one approach is always faster.

---

# Generics vs Interfaces

| Use Case | Generics | Interfaces |
|-----------|-----------|------------|
| Generic algorithms | ✅ | ❌ |
| Collections | ✅ | ❌ |
| Containers | ✅ | ❌ |
| Behavior abstraction | ❌ | ✅ |
| Dependency Injection | ❌ | ✅ |
| Mocking | ❌ | ✅ |

---

# Summary

Generics are ideal when writing the same algorithm for multiple data types.

Use them for:

- Collections
- Algorithms
- Utility functions
- Reusable data structures

Use interfaces when you want different types to expose the same behavior.

A simple rule of thumb:

> **Generics are for reusable data structures and algorithms. Interfaces are for reusable behavior.**
