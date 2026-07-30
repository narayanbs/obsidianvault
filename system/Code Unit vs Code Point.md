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