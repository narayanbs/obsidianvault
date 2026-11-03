
# Concepts 
### Predeclared types

```go
* bool
* int8/uint8, int16/uint16, int32/uint32, int64/uint64, int/uint, byte(uint8) 
* float32, float64, complex64, complex128
* string, rune(int32) 
```

### Type Literals

```go
* array, struct, pointer, function, slice, map, channel, interface
```


### Type Declaration - (Type Definition and Type Alias)

```go
type (
	alias = type
	typeName type
)
```

former is `type alias`, latter is `type definition`

* alias is just another name for the existing type
* typeName creates a new type with the same underlying type as the type referred

### Named Types and Unnamed Types

* `type definition` and `predeclared types` are called `named types`, if type alias refers to a named type it is also named
* `type literals` are `unnamed types`

**Each Named type has a unique identity.  Unnamed types are identical if they are structurally identical.**
### Underlying Type

**All types have an underlying type.**

* The underlying type of predeclared types and type literals are the types themselves.
* The underlying type of a type definition is the underlying type of the type it is referring to or if it is referring to a type literal, then the underlying type is the literal itself.


# Type Conversion

### Implicit and Explicit Conversion

* Implicit conversion is done through assignment
* Explicit is of the form `T(x)`, where `T` is the type and `x` is the value

## Rules

1. All types that are identical can be implicitly converted
```go
   int <-> int, uint8 <-> byte, []byte <-> []uint8, *int <-> *int 
```
      
2. if types share the same underlying type, they can be explicitly converted
```go
	type MyInt int

	var a MyInt	// a is a value of named type
	var b int	// b is a value of named type
	
the underlying type of MyInt and int are the same i.e int, so the values can be explicitly converted

	a = MyInt(b) 
	b = int(a)
```

3. if types share the same underlying type and one of them is an unnamed type they can be implicitly converted
```go
	type IntPtr *int 
	
	var a IntPtr 	// a is a value of Named type 
	var b *int 	// b is a value of unnamed type {type literal}
	
since the underlying type of both are the same, they can be implicitly converted.
	
	a = b	 
	b = a
```
4. **Pointer**
   In Go, an unnamed pointer type can be represented with `*T`, where T can be an arbitrary type.  Type `T` is called the `base type` of pointer type `*T`.
   
   Suppose we have 
```go
  type MyInt int64
  type Ta *int64
  type Tb *MyInt	  
```

* values of type `*int64` can be implicitly converted to type `Ta`, and vice versa, for their underlying types are both `*int64`.
 -  values of type `*MyInt` can be implicitly converted to type Tb, and vice versa, for their underlying types are both `*MyInt`.
 -  values of type `*MyInt` can be explicitly converted to type `*int64`, and vice versa, for they are both unnamed  and the underlying types of their base types are both int64.
   
   Example
```go
  var a *int64 
  var b *MyInt 
  a = (*int64)(b)
  or
  b = (*MyInt)(a)   
```

values of type `Ta` can't be directly converted to type `Tb`, even if explicitly. However, by the just listed first three facts,  a value `pa` of type `Ta` can be indirectly converted to type Tb by nesting three explicit conversions, `Tb((*MyInt)((*int64)(pa)))`.

5. **Interface**
   *  A value v with type `T` that implements interface `I`, then value can be implicitly converted to `I`      
   
6. **string**
   * A string value can be explicitly converted to a slice type whose underlying type is `[]byte (a.k.a., []uint8)`, and vice versa.
   * A string value can be explicitly converted to a slice type whose underlying type is `[]rune (a.k.a., []int32)`, and vice versa
   
7. **slice**
 * slice may be converted (explicitly) to an array. if the length of the array type is larger than the length of the slice, a panic occurs.
     
8. **channel**
   * bidirectional channel can be converted implicitly to another channel (bidirectional/non-bidirectional),if they have same element type and either one is an unnamed type. 
   
9. **Constants**
   * If `x` is an `untyped` constant representable by a value of type `T`, then `x` can be implicitly converted
   * If `x` is a `typed` constant representable as a value of type `T`, then x can be explicitly converted to `T`
     
```go
   // The implicit conversions are all legal.
	const I = 123		// untyped constant, whose default type is int
	const I1, I2 int8 = 0x7F, -0x80		// I1, I2 are typed constants 
	const I3, I4 int8 = I, 0.0			// I3, I4 are typed constants
	const F = 0.123456789	// Untyped constant, whose default type is float64
	const F32 float32 = F
	const F32b float32 = I
	const F64 float64 = F
	const C1, C2 complex64 = F, I
	
	// const F64b float64 = I3 // doesn't compile
	const F64b = float64(I3)   // compiles okay
	
	// const I5 int = C2 // doesn't compile
	const I5 = int(C2)   // compiles okay
```

10. **Non Constants**
    * Non-constant floating-point and integer values can be explicitly converted to any floating-point and integer types.
    * When converting a floating-point non-constant number to an integer, the fraction is discarded (truncation towards zero).
    * Data overflow and rounding are allowed in non-constant number conversions. 
    * Non-constant complex values can be explicitly converted to any complex types.
      
```go
package main

import "fmt"

func main() {
	var a, b = 1.6, -1.6 // both are float64
	fmt.Println(int(a), int(b)) // 1 -1

	var i, j int16 = 0x7FFF, -0x8000
	fmt.Println(int8(i), uint16(j)) // -1 32768

	var c1 complex64 = 1 + 2i
	var _ = complex128(c1)
}
```

11. **Function**
    ```go
    import "fmt"

	type Greeting func(name string) string

	func (g Greeting) exclamation(name string) string {
		return g(name) + "!"
	
	func english(name string) string { return "Hello, " + name }

	func main() {
		fmt.Println(emglish("Alisus"))
		fmt.Println(Greeting(english).exclamation("Alisus"))

			or

		var g Greeting
		g = english
		fmt.Println(g("Alisus"))

	}

	or

	import "fmt"

	type Greeting struct {
	    say func(name string) string
	}

	func newGreeting(f func(string) string) *Greeting {
	    return &Greeting{say: f}
	}

	func (g *Greeting) exclamation(name string) string { return g.say(name) + "!" }

	func main() {
	    english := &Greeting{say: func(name string) string {
		return "Hello, " + name
	    }}

	    french := newGreeting(func(name string) string {
		return "Bonjour, " + name
	    })

	    fmt.Println(english.exclamation("ANisus"))
	    fmt.Println(french.exclamation("ANisus"))
	}
    ```


### Examples

```go
1) Basic 
   
package main

func main() {
	type IntSlice []int
	type MySlice []int

	var is = IntSlice{}
	var ms = MySlice{}

	is = IntSlice(ms)
	ms = MySlice(is)

	type Foo = struct {
		n int `name:"foo"`
	}
	type Bar = struct {
		n int `name:"bar"`
	}

	var x map[Foo]Bar
	var y map[Bar]Foo

	x = map[Foo]Bar(y)
	y = map[Bar]Foo(x)

	var s = []int{}

	s = is
	s = ms
	is = s
	ms = s

}

2) Pointer related

package main

func main() {
	type MyInt int
	type IntPtr *int
	type MyIntPtr *MyInt

	var pi = new(int)

	var ip IntPtr = pi

	// var _ *MyInt = pi
	var _ *MyInt = (*MyInt)(pi)

	//
	// var _ MyIntPtr = pi
	// var _ = MyIntPtr(pi)
	var _ MyIntPtr = (*MyInt)(pi)
	var _ = MyIntPtr((*MyInt)(pi))

	//
	// var _ MyIntPtr = ip
	// var _  = MyIntPtr(ip)
	var _ MyIntPtr = (*MyInt)((*int)(ip))
	var _ = MyIntPtr((*MyInt)((*int)(ip)))

}

3) Channel

package main

func main() {
	type C chan string
	type C1 chan<- string
	type C2 <-chan string

	var ca C
	var cb chan string

	ca = cb
	cb = ca

	var _, _ chan<- string = ca, cb
	var _, _ <-chan string = ca, cb
	var _ C1 = cb
	var _ C2 = cb

	// Can't be converted directly
	//
	// var _ = C1(ca)
	// var _ = C2(ca)
	//

	var _ C1 = (chan<- string)(ca)
	var _ C2 = (<-chan string)(ca)

	var _ = C1((chan<- string)(ca))
	var _ = C2((<-chan string)(ca))

}
```
