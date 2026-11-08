
In Go, when you declare a variable using `:=` with an integer literal, Go defaults the type to `int`, regardless of how large the value is. It does not automatically switch to `int64`.

For example:

```go
a := 100
fmt.Printf("%T\n", a) // int
```

Even with a large integer:

```go
a := 2147483648
fmt.Printf("%T\n", a) // int
```

On a 64-bit system, this works because `int` is 64 bits.

However, on a 32-bit system, this will result in a compile-time error because the value cannot fit in an `int`.

### How to explicitly declare int64

```go
var a int64 = 2147483648
```

Or:

```go
a := int64(2147483648)
```


One interesting Go concept: Integer constants are initially untyped. This means they can represent values larger than `int64` too, as long as they are within Go's constant representation limits.

For example:

```go
const x = 9223372036854775808
```

This is valid, even though it exceeds the maximum `int64` value. But assigning it to an `int64` variable would produce a compile-time error.

Similarly when we write

```go
a := 1.234
```

The inferred type of `a` is `float64`.

You can verify it:

```go
package main

import "fmt"

func main() {
    a := 1.234
    fmt.Printf("%T\n", a) // float64
}
```

Go's default types for untyped numeric constants are:

| Constant | Default type |
| -------- | ------------ |
| `123`    | `int`        |
| `1.234`  | `float64`    |
| `1 + 2i` | `complex128` |
# Converting Between Integer Types

Go has many integer data types to pick from. Go sometimes automatically generates numeric values as `int`, which may not match your input value. If your input value were `int64`, you would not be able to use the `int` and the `int64` numbers in the same mathematical expression until you converted their data types to match.


### Smaller Integer to Larger Conversion


```go
var index int8 = 15
var bigIndex int32
bigIndex = int32(index)
fmt.Println(bigIndex)

// Output
// 15

fmt.Printf("index data type:    %T\n", index)
fmt.Printf("bigIndex data type: %T\n", bigIndex)

// Output
// index data type:    int8
// bigIndex data type: int32
```

### Larger to Smaller Conversion


```go
var big int64 = 64
var little int8
little = int8(big)
fmt.Println(little)

// Output
// 64
```

### Wrap-Around

Keep in mind that when converting integers you could potentially exceed the maximum value of the data type and overflow. When this occurs, the value may wrap to become a very small or negative number.


```go
var big int64 = 129
var little = int8(big)
fmt.Println(little)

// Output
// -127
```

> **Why?** Because $129 = 127 + 2$, so $127 + 1$ is `-128` and $127 + 2$ is `-127`.
> 
>   

# Converting Integers to Floats


```go
var x int64 = 57
var y float64 = float64(x)
fmt.Printf("%.2f\n", y)

// Output
// 57.00
```

# Converting Floats to Integers

The conversion involves dropping the decimal place.

  
```go
// Compile-time constant conversion
// var a int = int(1.23452) // Compilation error

// Runtime conversion
var b float64 = 1.23452
var c int = int(b)
fmt.Printf("b = %.2f\n", b)
fmt.Printf("c = %d\n", c)

var f float64 = 390.8
var i int = int(f)
fmt.Printf("f = %.2f\n", f)
fmt.Printf("i = %d\n", i)

// Output
// b = 1.23
// c = 1
// f = 390.80
// i = 390
```

# Numbers Converted Through Division

If, when dividing, any of the number types are a float, then all of the types will automatically be declared as a float:


```go
a := 5.0 / 2
// or 5 / 2.0 
fmt.Println(a)

// Output
// 2.5
```

When dividing integer types in Go the result will also be an integer type:

  
```go
a := 5 / 2
fmt.Println(a)

// Output
// 2
```

# Common Conversions

The most common numeric conversions are `Atoi` (string to int) and `Itoa` (int to string).

```go
i, err := strconv.Atoi("-42")
s := strconv.Itoa(-42)
```

### Conversion of Values to Strings


```go
s := strconv.FormatBool(true)
s := strconv.FormatFloat(3.1415, 'E', -1, 64)
s := strconv.FormatInt(-42, 16)
s := strconv.FormatUint(42, 16)
```

### Conversion of Strings to Values


```go
b, err := strconv.ParseBool("true")
f, err := strconv.ParseFloat("3.1415", 64)
i, err := strconv.ParseInt("-42", 10, 64)
u, err := strconv.ParseUint("42", 10, 64)
```

# Examples

### Integers to Strings

Use `strconv.Itoa` to get the decimal string representation of an integer.

  
```go
strconv.Itoa(97) // "97"
```

**Example:**

```go
times := 5
fmt.Println("hello number is " + times)

// Output
// invalid operation: "hello number is " + times (mismatched types untyped string and int)

fmt.Println("hello number is " + strconv.Itoa(times))
```


```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    a := strconv.Itoa(12)
    fmt.Printf("%q\n", a)
}

// Output
// "12"
```

####  using string(....) is different 

Here, the value is interpreted as a Unicode code point, and the resulting string will contain the character represented by that code point, encoded in UTF-8.
  
```go
string(97) // "a"
```

If the value does not represent a valid code point (for instance if it’s negative), the result will be `"\ufffd"`, the Unicode replacement character .


```go
string(-1) // "\ufffd" == "\xef\xbf\xbd"
```


---

### Float to String


Use `strconv.FormatFloat` (Recommended for Performance)

**Syntax:**

```go
s := strconv.FormatFloat(f, fmt, prec, bitSize)

```

* **`f`**: The float value (`float64`).
* **`fmt`**: The format byte (`'f'` for standard decimal, `'e'` for scientific notation, `'g'` for the most compact representation).
* **`prec`**: The precision (number of digits after the decimal point, or `-1` to use the minimum number of digits necessary to accurately represent the float).
* **`bitSize`**: The original float size (`32` for `float32`, `64` for `float64`).

**Example:**

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    var f float64 = 3.14159265

    // Format as standard decimal with 2 decimal places
    s1 := strconv.FormatFloat(f, 'f', 2, 64)
    fmt.Println(s1) // Output: "3.14"

    // Format using scientific notation
    s2 := strconv.FormatFloat(f, 'e', -1, 64)
    fmt.Println(s2) // Output: "3.14159265e+00"
}

```

---

Use `fmt.Sprintf` (Easiest & Most Flexible)

**Example:**

```go
package main

import "fmt"

func main() {
    var f float64 = 55.6789

    // Format to 2 decimal places
    s := fmt.Sprintf("%.2f", f)
    fmt.Println(s) // Output: "55.68" (note: it rounds automatically)

    // Standard float representation
    sDefault := fmt.Sprintf("%f", f)
    fmt.Println(sDefault) // Output: "55.678900"
}

```

### Why use strconv.FormatFloat instead of fmt.Sprintf?

While `fmt.Sprintf("%f", val)` is more concise, it uses reflection underneath. If you are working on high-performance applications, serializing large amounts of data, or looping over thousands of rows, `strconv.FormatFloat` is roughly three times faster and prevents unnecessary memory allocation

---




```go
package main

import (
    "fmt"
)

func main() {
    lines_yesterday := "50"
    lines_today := "108"

    lines_more := lines_today - lines_yesterday

    fmt.Println(lines_more)
}

// Output
// invalid operation: lines_today - lines_yesterday (operator - not defined on string)
```

### Strings to Integers

To convert strings to integers, we use `strconv.Atoi`:

  

```go
package main

import (
    "fmt"
    "log"
    "strconv"
)

func main() {
    lines_yesterday := "50"
    lines_today := "108"

    yesterday, err := strconv.Atoi(lines_yesterday)
    if err != nil {
        log.Fatal(err)
    }

    today, err := strconv.Atoi(lines_today)
    if err != nil {
        log.Fatal(err)
    }
    lines_more := today - yesterday

    fmt.Println(lines_more)
}

// Output
// 58
```

### Try Converting Something That Is Not a Number

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    a := "not a number"
    b, err := strconv.Atoi(a)
    fmt.Println(b)
    fmt.Println(err)
}

// Output
// 0
// strconv.Atoi: parsing "not a number": invalid syntax
```

### Strings to Floats

We use `strconv.ParseFloat`:


```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    fStr := "3.14159265"

    // Convert to float32
    if s, err := strconv.ParseFloat(fStr, 32); err == nil {
        fmt.Println(s) // 3.1415927410125732 (float32 precision)
    }

    // Convert to float64 (recommended for maximum precision)
    if s, err := strconv.ParseFloat(fStr, 64); err == nil {
        fmt.Println(s) // 3.14159265 (full float64 precision)
    }
}

```

---

### Handling Errors

Because strings can contain malformed or non-numeric data, `ParseFloat` returns an error value. Always handle the error to prevent runtime crashes if the input string is invalid:

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    invalidStr := "not-a-number"

    val, err := strconv.ParseFloat(invalidStr, 64)
    if err != nil {
        fmt.Println("Conversion failed:", err)
        return
    }

    fmt.Println(val)
}

```

## Strings and Byte Slices

- Converting a slice of bytes to a string type yields a string whose successive bytes are the elements of the slice.
    
    
    ```go
    string([]byte{97, 230, 151, 165}) // "a日"
    ```
    
- Converting a value of a string type to a slice of bytes type yields a slice whose successive elements are the bytes of the string.
    
    
    ```go
    []byte("a日") // []byte{97, 230, 151, 165}
    ```
    

## Strings and Rune Slices

- Converting a slice of runes to a string type yields a string that is the concatenation of the individual rune values converted to strings.
        
    
    ```go
    string([]rune{97, 26085}) // "a日"
    ```
    
- Converting a value of a string type to a slice of runes type yields a slice containing the individual Unicode code points of the string.
    
    ```go
    []rune("a日") // []rune{97, 26085}    
    ```
    

# Underlying Type

A non-constant value can be converted to type `T` if it has the same underlying type as `T`.
In this example, the underlying type of `int64`, `T1`, and `T2` is `int64`.

```go
type (
    T1 int64
    T2 T1
)
```

It’s idiomatic in Go to convert the type of an expression to access a specific method.

```go
var n int64 = 12345
fmt.Println(n)              // 12345
fmt.Println(time.Duration(n)) // 12.345µs
```

_(The underlying type of `time.Duration` is `int64`, and the `time.Duration` type has a `String` method that returns the duration formatted as a time.)_

# Implicit Conversions

The only implicit conversion in Go is when an untyped constant is used in a situation where a type is required.

In this example, the untyped literals `1` and `2` are implicitly converted:

```go
var x float64
x = 1 // Same as x = float64(1)

t := 2 * time.Second // Same as t := time.Duration(2) * time.Second
```

_(The implicit conversions are necessary since there is no mixing of numeric types in Go. You can only multiply a `time.Duration` with another `time.Duration`.)_


When the type can’t be inferred from the context, an untyped constant is converted to a `bool`, `int`, `float64`, `complex128`, `string`, or `rune` depending on the syntactical format of the constant.

```go
n := 1   // Same as n := int(1)
x := 1.0 // Same as x := float64(1.0)
s := "A" // Same as s := string("A")
c := 'A' // Same as c := rune('A')
```

Illegal implicit conversions are caught by the compiler:


```
var b byte = 256 // Same as var b byte = byte(256)
// ../main.go:2:6: constant 256 overflows byte
```

# Type Assertions and Type Switches

### Type Assertions

A type assertion doesn’t really convert an interface to another data type, but it provides access to an interface’s concrete value, which is typically what you want.

The type assertion `x.(T)` asserts that the concrete value stored in `x` is of type `T`, and that `x` is not `nil`.
- If `T` is not an interface, it asserts that the dynamic type of `x` is identical to `T`.
- If `T` is an interface, it asserts that the dynamic type of `x` implements `T`.
    
```go
var x interface{} = "foo"

var s string = x.(string)
fmt.Println(s)      // "foo"

s, ok := x.(string)
fmt.Println(s, ok) // "foo true"

n, ok := x.(int)
fmt.Println(n, ok) // "0 false"

n = x.(int)        // ILLEGAL
// panic: interface conversion: interface {} is string, not int
```

### Type Switches

A type switch performs several type assertions in series and runs the first case with a matching type.

```go
var x interface{} = "foo"

switch v := x.(type) {
case nil:
    fmt.Println("x is nil")            // here v has type interface{}
case int: 
    fmt.Println("x is", v)             // here v has type int
case bool, string:
    fmt.Println("x is bool or string") // here v has type interface{}
default:
    fmt.Println("type unknown")        // here v has type interface{}
}

// Output
// x is bool or string
```