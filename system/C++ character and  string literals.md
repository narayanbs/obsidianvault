
###  Character literal

 A **character literal** represents a single character/code unit and is written with **single quotes**:

```
'A'
'7'
'\n'
'\u03A9'   // Ω
```

 Its type is generally `char` for an ordinary character literal.

 There are also prefixed character literals:

```
u8'a'   // char8_t
u'a'    // char16_t
U'a'    // char32_t
L'a'    // wchar_t
```

Let's consider 
```cpp
char c = '\u03A9'   // Ω
```

`\u03A9'` is a **character literal** representing the Unicode character **Ω (Greek capital letter Omega)**.

However, there’s an important detail:

- Its type is `char` **if the character can be represented in the execution character set**.
- Otherwise, depending on the literal/context and encoding, you may need a wider character type such as `wchar_t`, `char16_t`, or `char32_t`.

`char c = '\u03A9';`  Whether this is valid and what value c gets depends on the implementation's character set/encoding.

 If you encode Ω (U+03A9) as UTF-8, it becomes two bytes:
 ```cpp
 Ω
Unicode: U+03A9
UTF-8:   CE A9
 ```
 A `char` holds only one byte, whereas UTF-8 may require multiple bytes for a single Unicode character.
Suppose a C++ implementation uses ISO-8859-7 for its execution character set:
ISO-8859-7 assigns Ω (U+03A9) to byte: `D9`
so conceptually
```cpp
'\u03A9'  →  Ω  →  execution character set → 0xD9
```
and `char c = '\u03A9';`  can result in `c == '\xD9'`


### String literal

 A **string literal** represents a sequence of characters/code units and is written with **double quotes**:

```
"Hello"
"ABC"
"Hello\n"
```

 For example:

```
"ABC"
```

 contains the three visible characters `A`, `B`, `C`, **plus a terminating null character** `'\0'`.

 There are also prefixed string literals:

```
u8"Hello"   // UTF-8
u"Hello"    // UTF-16
U"Hello"    // UTF-32
L"Hello"    // wide string
```

---

 ## What if one Unicode character needs multiple code units?

 For example, in UTF-8, the character **😀** has code point:

```
U+1F600
```

 but its UTF-8 representation requires **4 code units**:

```
F0 9F 98 80
```

This is the important distinction:

 > **A Unicode code point is not necessarily one C++ character/code unit.**

 So what happens with:

```
u8"😀"
```

 That's perfectly fine. The string literal contains **4 UTF-8 code units**, followed by `'\0'`.

 Conceptually:

```
u8"😀"
     ↓
[F0] [9F] [98] [80] [00]
```

 The type is an array of `char8_t` (in modern C++).

 ### But what about a character literal?

 This is where things get interesting.

 You **cannot generally put a multi-code-unit UTF-8 character into an ordinary character literal**:

```
'😀'       // not a UTF-8 character literal representing 4 code units
```

 A character literal is designed around a **single code unit/value**, depending on its prefix. It cannot simply become "four `char`s" because you wrote one character between `' '`.

 For example:

```
u8'a'      // one char8_t
u'😀'      // one char16_t? No: 😀 doesn't fit in one UTF-16 code unit
U'😀'      // one char32_t, works
```

 `U'😀'` works because `char32_t` is capable of representing the Unicode code point `U+1F600` in a single code unit.

### So what happens to  `char c = '😀';`

`char c = '😀';` is **not a valid way to store 😀 in a `char`**.

 There are two separate issues:

```
char c = '😀';
```

 ### 1\. `'😀'` is a character literal

 The compiler has to interpret the contents of `'...'` as a character literal. But 😀 (`U+1F600`) cannot be represented by a single ordinary `char` in UTF-8.

 A `char` is only **one byte/code unit**. 😀 in UTF-8 needs **4 bytes**:

```
😀
↓
F0 9F 98 80
```

 So one `char` cannot hold the UTF-8 representation of 😀.

 ### 2\. What does the compiler actually do?

 Don't think of it as:

```
😀 → somehow take first byte → c
```

 Instead, the character literal itself has language-defined rules, and a multi-byte character that doesn't fit the literal's encoding can make the program **ill-formed or implementation-dependent**, depending on the exact literal/encoding and compiler.

 In practice, you should **not write**:

```
char c = '😀';   // ❌ don't use this
```

 If you want the Unicode **code point** itself, use `char32_t`:

```
char32_t c = U'😀';  // ✅ c contains U+1F600
```

 If you want its **UTF-8 encoding**, use a string:

```
const char* s = u8"😀";  // UTF-8: 4 code units
```

 Or, in modern C++:

```
const char8_t* s = u8"😀";
```

 So the mental model is:

```
U'😀'       → one char32_t code unit → U+1F600
u8"😀"      → four char8_t code units → F0 9F 98 80
char c      → one byte → cannot hold the UTF-8 encoding of 😀
```

 **Most important:** a C++ `char` is **not necessarily a Unicode character**. It is fundamentally a small integer type commonly used as a **byte/code unit**.


# Big Picture


 ## 1\. The big picture

 There are three different things to keep in mind:

```
Unicode code point
        ↓
     encoding
        ↓
   code units
        ↓
C++ character/string objects
```

 For example, the Unicode character 😀 is the code point:

```
U+1F600
```

 It can be encoded as:

```
UTF-8   → F0 9F 98 80       → 4 × 8-bit code units
UTF-16  → D83D DE00         → 2 × 16-bit code units
UTF-32  → 0001F600          → 1 × 32-bit code unit
```

 That's the foundation.

---

 # 2\. `char`

 In C and C++:

```
char c;
```

 A `char` is exactly **one byte**:

```
sizeof(char) == 1
```

 But be careful: **one byte does not necessarily mean 8 bits according to the C/C++ language standards.**

 A byte is defined as the size of a `char`, and `CHAR_BIT` tells you how many bits it contains. On essentially all modern systems:

```
CHAR_BIT = 8
```

 so we normally say:

 > `char` is an 8-bit byte.

 Historically, `char` was commonly used for ordinary characters from the execution character set, often ASCII-compatible.

 But:

 > **`char` does not inherently mean ASCII.**

 It is fundamentally a small integer type capable of representing a byte.

---

 # 3\. Character literal `'A'`

 This is an important distinction.

 In **C**:

```
'A'
```

 is a **character constant**, and its type is `int`.

 In **C++**:

```
'A'
```

 is a **character literal**, and its type is `char`.

 So:

```
char c = 'A';
```

 In C:

```
'A' → int → converted to char
```

 In C++:

```
'A' → char
```

 This is one of the differences between C and C++.

---

 # 4\. String literal `"ABC"`

 A string literal is different from a character literal.

```
"ABC"
```

 is a sequence of characters with a terminating null character:

```
'A' 'B' 'C' '\0'
```

 In C++:

```
const char* p = "ABC";
```

 The literal is an array:

```
const char[4]
```

 Conceptually:

```
+---+---+---+----+
| A | B | C | \0 |
+---+---+---+----+
```

 So:

```
'A'     // one character literal
"ABC"   // string literal
```

 are fundamentally different things.

---

 # 5\. Wide characters: `wchar_t`

 C and C++ also have:

```
wchar_t
```

 and the corresponding wide character literal:

```
L'A'
```

 and wide string:

```
L"hello"
```

 For example:

```
wchar_t c = L'A';
```

 The important thing is:

 > `wchar_t` is intended to represent a wide character, but **its size and encoding are implementation/platform dependent**.

 Typically:

```
Windows:
wchar_t = 16 bits

Linux/macOS:
wchar_t = 32 bits
```

 And therefore you should **not think of `wchar_t` as "UTF-16" or "UTF-32" in C++**.

 On Windows it is commonly used with UTF-16 semantics; on Unix-like systems it is commonly UTF-32-like.

 Your statement that it can hold "the largest character in the locale" is close to the historical purpose of `wchar_t`, but don't interpret that as a universal Unicode guarantee.

---

 # 6\. Unicode code points

 Unicode assigns each character/code point a number.

 For example:

```
'A'       → U+0041
'é'       → U+00E9
'中'      → U+4E2D
'😀'      → U+1F600
```

 These are **code points**.

 A code point is not the same thing as a byte.

 For example:

```
😀
U+1F600
```

 is one Unicode code point.

 But its UTF-8 representation is:

```
F0 9F 98 80
```

 which consists of four bytes/code units.

---

 # 7\. UTF-8, UTF-16 and UTF-32

 These are **encodings of Unicode code points**.

 ### UTF-8

 Uses 8-bit code units:

```
1–4 code units per code point
```

 Example:

```
A     → 41
é     → C3 A9
😀    → F0 9F 98 80
```

 ### UTF-16

 Uses 16-bit code units:

```
1–2 code units per code point
```

 Example:

```
A     → 0041
é     → 00E9
😀    → D83D DE00
```

 ### UTF-32

 Uses 32-bit code units:

```
1 code unit per code point
```

 Example:

```
A     → 00000041
é     → 000000E9
😀    → 0001F600
```

---

 # 8\. `char16_t` — C++11

 C++11 introduced:

```
char16_t
```

 which is a distinct type intended for **UTF-16 code units**.

 For example:

```
char16_t c = u'A';
```

 And:

```
u"hello"
```

 is a UTF-16 string literal.

 You can have:

```
const char16_t* s = u"hello";
```

 Important:

 > `char16_t` represents **one UTF-16 code unit**, not necessarily one Unicode code point.

 Therefore 😀 needs two:

```
😀
   ↓ UTF-16
D83D DE00
↑    ↑
2 char16_t code units
```

---

 # 9\. `char32_t` — C++11

 C++11 also introduced:

```
char32_t
```

 which is intended for UTF-32 code units.

 For example:

```
char32_t c = U'😀';
```

 and:

```
U"hello 😀"
```

 is a UTF-32 string literal.

 Because UTF-32 uses one 32-bit code unit per Unicode code point:

```
😀 → 0001F600
     ↑
 one char32_t
```

 So `char32_t` is particularly convenient when you want a type that can hold a Unicode code point directly.

 But remember that **Unicode code point ≠ user-perceived character**. Some visible characters/graphemes can consist of multiple code points.

---

 # 10\. `char8_t` — C++20

 C++20 introduced:

```
char8_t
```

 specifically for **UTF-8 code units**.

 For example:

```
char8_t c = u8'A';
```

 and:

```
u8"hello"
```

 is a UTF-8 string literal.

 For 😀:

```
😀
 ↓ UTF-8

F0 9F 98 80
↑  ↑  ↑  ↑
4 char8_t code units
```

 So:

```
char8_t
```

 does **not** mean "a UTF-8 character."

 It means:

 > **one UTF-8 code unit.**

---

 # 11\. The character literal prefixes

 Modern C++ therefore gives you these forms:

```
'A'       // ordinary character literal
L'A'      // wide character literal
u'A'      // char16_t character literal
U'A'      // char32_t character literal
u8'A'     // char8_t character literal (C++20)
```

 Conceptually:

| Literal | Type          | Intended encoding                    |
| ------- | ------------- | ------------------------------------ |
| `'A'`   | `char` in C++ | execution character set              |
| `L'A'`  | `wchar_t`     | implementation-defined wide encoding |
| `u'A'`  | `char16_t`    | UTF-16                               |
| `U'A'`  | `char32_t`    | UTF-32                               |
| `u8'A'` | `char8_t`     | UTF-8                                |

 There are some additional rules around character literals containing characters that require multiple code units, so don't take "one literal = one code unit" as universally true for every possible literal. But this table is the right conceptual model.

---

 # 12\. The corresponding string literals

 The same prefixes apply to strings:

```
"hello"
L"hello"
u"hello"
U"hello"
u8"hello"
```

 Conceptually:

| String literal | Element type | Encoding                    |
| -------------- | ------------ | --------------------------- |
| `"hello"`      | `char`       | ordinary/execution encoding |
| `L"hello"`     | `wchar_t`    | implementation-defined      |
| `u"hello"`     | `char16_t`   | UTF-16                      |
| `U"hello"`     | `char32_t`   | UTF-32                      |
| `u8"hello"`    | `char8_t`    | UTF-8                       |

 For example:

```
const char*     a = "hello";
const wchar_t*  b = L"hello";
const char16_t* c = u"hello";
const char32_t* d = U"hello";
const char8_t*  e = u8"hello";
```

---

 # 13\. The most important terminology

 This is probably the part worth memorizing:

 ### Code point

 A number assigned by Unicode:

```
😀 → U+1F600
```

 ### Code unit

 The basic storage unit used by an encoding.

```
UTF-8  → 8-bit code unit
UTF-16 → 16-bit code unit
UTF-32 → 32-bit code unit
```

 Therefore:

```
UTF-8:
😀 → F0 9F 98 80
     └── 4 code units ──┘

UTF-16:
😀 → D83D DE00
     └─ 2 code units ─┘

UTF-32:
😀 → 0001F600
     └─ 1 code unit ──┘
```

---

 # 14\. One important correction to your original understanding

 You said:

 > "wide character can hold the largest character in the locale"

 I'd modify that to:

 > **`wchar_t` is a wide-character type whose size and representation are implementation-defined; historically it was intended to represent characters from the implementation's wide-character/locale system. It should not be assumed to mean Unicode, UTF-16, or UTF-32.**

 Whereas:

```
char8_t   → UTF-8 code unit
char16_t  → UTF-16 code unit
char32_t  → UTF-32 code unit
```

 are much more explicitly tied to Unicode encodings.

---

 # 15\. The whole picture

 You can keep this mental model:

```
                         Unicode
                            │
                     ┌──────┴──────┐
                     │             │
                 Code point    e.g. U+1F600
                     │
             choose an encoding
                     │
       ┌─────────────┼─────────────┐
       │             │             │
     UTF-8         UTF-16        UTF-32
       │             │             │
    8-bit CU      16-bit CU     32-bit CU
       │             │             │
   char8_t        char16_t      char32_t
       │             │             │
   u8"..."         u"..."        U"..."
```

 And separately:

```
Traditional C/C++
       │
       ├── char
       │     └── ordinary/narrow character or byte
       │
       └── wchar_t
             └── implementation-defined wide character
```

 So the **modern C++ character landscape** is essentially:

```
char       → ordinary byte/character
wchar_t    → implementation-defined wide character

char8_t    → UTF-8 code unit
char16_t   → UTF-16 code unit
char32_t   → UTF-32 code unit
```

 while the literals are:

```
'A'       L'A'       u'A'       U'A'       u8'A'
 │          │          │          │           │
char      wchar_t   char16_t   char32_t    char8_t
```

 and strings:

```
"..."     L"..."     u"..."     U"..."     u8"..."
 │          │          │          │           │
char[]    wchar_t[] char16_t[] char32_t[] char8_t[]
```

 **One final subtlety:** a Unicode **code point is not necessarily what a user thinks of as one "character."** For example, `é` can be represented as either one code point (`U+00E9`) or as `e` \+ a combining acute accent (two code points). So the hierarchy is really:

```
user-perceived character (grapheme)
             ↓
       one or more code points
             ↓
       one or more code units
             ↓
            bytes
```

 That distinction becomes very important once you start dealing seriously with Unicode in C++.




The distinction between a **code point** and a **code unit** is important.

- A **Unicode code point** is an abstract number assigned to a character, such as:
    - `'A'` → `U+0041`
    - `'€'` → `U+20AC`
    - `'😀'` → `U+1F600`
- A **UTF-16 code unit** is a **16-bit value** used by the UTF-16 encoding.

The relationship is:

- If the code point is in the **Basic Multilingual Plane (BMP)** (`U+0000` to `U+FFFF`, excluding surrogate values), it is encoded as **one UTF-16 code unit**.
- If the code point is above `U+FFFF` (`U+10000` to `U+10FFFF`), it is encoded as **two UTF-16 code units**, called a **surrogate pair**.

### Examples

#### `'A'`

Code point:
```
U+0041
```
UTF-16 encoding:
```
0x0041
```
One code point → one `char16_t`.
```c++
char16_t c = u'A';   // 0x0041
```
#### `'€'`

Code point:
```
U+20AC
```
UTF-16 encoding:
```
0x20AC
```
Again, one code point → one `char16_t`.
#### `'😀'`

Code point:
```
U+1F600
```
UTF-16 encoding:
```
0xD83D 0xDE00
```
Here there are **two UTF-16 code units**.
```c++
char16_t s[] = u"😀";
```
The array contains
```
0xD83D
0xDE00
0x0000   // null terminator
```
Notice that **no single `char16_t` contains the code point `U+1F600`**. Each `char16_t` contains one half of the surrogate pair.

---

By contrast, `char32_t` is designed so that **one `char32_t` can represent one Unicode code point**:
```c++
char32_t c = U'😀';   // value is 0x1F600
```
So:

- `char16_t` stores **UTF-16 code units**, not necessarily complete code points.
- `char32_t` stores **UTF-32 code units**, where each code unit is effectively one Unicode code point (for valid Unicode scalar values).

This is why the C++ standard describes `char16_t` as a type for **UTF-16 code units** rather than for Unicode characters or code points.


# char16_t and char 32_t in c++

Neither `char16_t` nor `char32_t` is considered signed or unsigned in the same sense as the integer types. They are **distinct fundamental types** introduced specifically to represent UTF-16 and UTF-32 code units.

Some details:

- `char16_t`
    
    - Size: **16 bits** (exactly 16 bits).
        
    - Intended to hold a UTF-16 code unit.
        
    - Range: `0` to `65535` (`0x0000` to `0xFFFF`).
        
    - Behaves as a non-negative integer type.
        
- `char32_t`
    
    - Size: **32 bits** (exactly 32 bits).
        
    - Intended to hold a UTF-32 code unit.
        
    - Range: `0` to `4294967295` (`0x00000000` to `0xFFFFFFFF`).
        
    - Behaves as a non-negative integer type.
        

Although they only represent non-negative values, they are **not aliases for `uint16_t` or `uint32_t`**. They are separate types, so for example:

```cpp
char16_t c16 = u'A';
char32_t c32 = U'A';

static_assert(!std::is_same_v<char16_t, uint16_t>);
static_assert(!std::is_same_v<char32_t, uint32_t>);
```

You can verify their properties using the type traits library:

```cpp
#include <type_traits>

static_assert(std::is_integral_v<char16_t>);
static_assert(std::is_integral_v<char32_t>);

static_assert(!std::is_signed_v<char16_t>);
static_assert(!std::is_signed_v<char32_t>);

static_assert(std::is_unsigned_v<char16_t>);
static_assert(std::is_unsigned_v<char32_t>);
```

So, in modern C++, `std::is_signed_v<char16_t>` is `false` and `std::is_unsigned_v<char16_t>` is `true` (likewise for `char32_t`). This reflects that they are integral types with only non-negative values, while still remaining distinct types rather than typedefs of the unsigned integer types.

This differs from plain `char`, whose signedness is implementation-defined (`char` may be signed or unsigned depending on the compiler and target).


# Summary

Here is a comprehensive summary of characters, character literals, and strings in C and C++, breaking down how the language handles standard, wide, and modern Unicode types.

---

### 1. Character Types and Sizes

C and C++ provide a variety of character types to handle everything from basic ASCII to global Unicode text.

| Type | Minimum Size | Primary Use Case | Literal Prefix / Example |
| --- | --- | --- | --- |
| **`char`** | 8 bits | Standard text, ASCII, narrow strings | None (`'a'`) |
| **`wchar_t`** | Implementation-defined (16 or 32 bits) | Wide characters / System locale characters | `L` (e.g., `L'a'`) |
| **`char8_t`** *(C++20)* | 8 bits | UTF-8 code units | `u8` (e.g., `u8'a'`) |
| **`char16_t`** *(C++11)* | 16 bits | UTF-16 code units | `u` (e.g., `u'a'`) |
| **`char32_t`** *(C++11)* | 32 bits | UTF-32 code units | `U` (e.g., `U'a'`) |

---

### 2. Character Literals vs. Integer Types

* **In C:** A character literal (e.g., `'a'`) has the type `int`. This is a historical quirk from early C. (Though a `sizeof('a')` is still `1` in C++, in C it evaluates to `sizeof(int)`).
* **In C:** String literals (e.g., `"abc"`) are arrays of type `char`.
* **In C++:** A narrow character literal (e.g., `'a'`) has the type **`char`**.
* **Type Safety:** Modern C++ types (`char8_t`, `char16_t`, `char32_t`) are distinct, fundamental types, preventing accidental mixing of UTF-8 data with standard ASCII `char` data.

---

### 3. Unicode and Modern Additions

As text encoding evolved beyond simple ASCII and local code pages, C++ introduced explicit support for modern Unicode standards:

* **Wide Characters (`wchar_t`):** Dependent on the platform (16-bit on Windows, 32-bit on Linux/macOS). It holds characters for the system's current locale, making it less portable for cross-platform Unicode.
* **UTF-8 (`char8_t` in C++20):** Designed specifically to store UTF-8 code units. UTF-8 is variable-length (1 to 4 bytes per character) and fully backward-compatible with ASCII.
* **UTF-16 (`char16_t` in C++11):** Stores UTF-16 code units (1 or 2 code units per character). Commonly used in Windows APIs and JavaScript interoperability.
* **UTF-32 (`char32_t` in C++11):** Fixed-length encoding where every single code point takes exactly 32 bits (4 bytes).

---

### 4. Character Strings and Prefixes

Strings follow the same prefix rules as individual character literals, always terminated by a null character (`\0` of the corresponding type).

* **Narrow String:** `"Hello"` (Type: `const char[]`)
* **Wide String:** `L"Hello"` (Type: `const wchar_t[]`)
* **UTF-8 String:** `u8"Hello"` (Type: `const char8_t[]` in C++20; `const char[]` prior to C++20)
* **UTF-16 String:** `u"Hello"` (Type: `const char16_t[]`)
* **UTF-32 String:** `U"Hello"` (Type: `const char32_t[]`)