
**`char8_t` is specifically intended to represent a UTF-8 code unit** in C++.

### What is a UTF-8 code unit?

 UTF-8 represents Unicode characters using **1 to 4 bytes**. Each individual byte in that encoding is called a **code unit**.

 For example, the character:

```
A
```

 has Unicode code point `U+0041` and its UTF-8 representation is:

```
41
```

 So it has **one UTF-8 code unit**.

 But:

```
€
```

 has Unicode code point `U+20AC`, whose UTF-8 encoding is:

```
E2 82 AC
```

 That's **3 UTF-8 code units**.

 And:

```
😀
```

 has code point `U+1F600`, encoded as:

```
F0 9F 98 80
```

 That's **4 UTF-8 code units**.

### Note

 `U+1F600`, is encoded in **UTF-16** as 
```
D83D DE00
```
That's **2 UTF-16 code units** = 4 bytes.


### Bottom Line
~~~
UTF-8: 1 code unit = 1 byte (8 bits)
UTF-16: 1 code unit = 2 bytes (16 bits)
UTF-32: 1 code unit = 4 bytes (32 bits)
~~~

### Where does `char8_t` fit?

 C++20 introduced:

```
char8_t
```

 as a distinct type specifically for **UTF-8 code units**.

 For example:

```
char8_t c = u8'A';
```

 And a UTF-8 string can be written:

```
const char8_t* str = u8"Hello 😀";
```

 Conceptually:

```
u8"😀"
   ↓
F0 9F 98 80
   ↓
4 char8_t code units
```

 
 > **`char8_t` represents a UTF-8 code unit, not a complete Unicode character/code point.**

 That's an important distinction.

| Concept            | Example       | Meaning                                       |
| ------------------ | ------------- | --------------------------------------------- |
| Unicode code point | `U+1F600`     | The abstract character/code point             |
| UTF-8 encoding     | `F0 9F 98 80` | Bytes used to encode that code point          |
| UTF-8 code unit    | `F0`          | One unit/byte of the UTF-8 encoding           |
| `char8_t`          | `char8_t`     | C++ type intended to hold one UTF-8 code unit |

One particularly important consequence is that **one `char8_t` does not necessarily correspond to one character**. A single Unicode code point may require 1–4 `char8_t`s.



# Golang rune 

**rune stores a Unicode code point**

Go:

```
rune // alias for int32 → Unicode code point
```

So
```
r := '💙'   // Unicode code point: U+1F499

fmt.Printf("%U\n", r) // U+1F499
fmt.Println(r)       // 128153 (in decimal)
```

**We can use char32_t  in c++ to store a unicode codepoint**
 So:

```
Go                    C++

rune                   char32_t
  ↓                       ↓
Unicode code point      Unicode code point
```


 For example, 😀 is Unicode code point `U+1F600`:

```
char32_t c = U'\U0001F600';  // one char32_t
```

 That's perfectly capable of representing 😀 because `char32_t` is 32 bits.

 Whereas:

```
char8_t
```

 is deliberately **not** intended to represent a Unicode character. It represents one unit of a UTF-8 encoding.

 
 Suppose you have:

```
😀
```

 Its Unicode code point is:

```
U+1F600
```

 A Go `rune` can hold that directly:

```
var r rune = '😀'
```

 Likewise in C++:

```
char32_t r = U'😀';
```

 But if you want its **UTF-8 representation**, you get:

```
F0 9F 98 80
```

 Those are four UTF-8 code units:

```
char8_t a = 0xF0;
char8_t b = 0x9F;
char8_t c = 0x98;
char8_t d = 0x80;
```

