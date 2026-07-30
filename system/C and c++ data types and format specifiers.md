# C/C++ Numeric Types: Literal Suffixes and `printf` Format Specifiers

> **Notes**
>
> - Integer literals are `int` by default unless a suffix changes the type.
> - Floating-point literals are `double` by default.
> - `printf` format specifiers are from the C standard library (`<stdio.h>` / `<cstdio>`).
> - `%u`, `%x`, `%o`, etc., print the **same type** in different representations.

| Type | Literal Suffix | Example | `printf` Format |
|------|----------------|---------|-----------------|
| `char` | *(none)* | `'A'` | `%c` |
| `signed char` | *(none)* | `(signed char)65` | `%hhd` |
| `unsigned char` | *(none)* | `(unsigned char)65` | `%hhu` |
| `short` | *(none)* | `(short)42` | `%hd` |
| `unsigned short` | *(none)* | `(unsigned short)42` | `%hu` |
| `int` | *(none)* | `42` | `%d` |
| `unsigned int` | `u` or `U` | `42u` | `%u` |
| `long` | `l` or `L` | `42L` | `%ld` |
| `unsigned long` | `ul`, `lu`, `UL`, `LU` | `42UL` | `%lu` |
| `long long` | `ll` or `LL` | `42LL` | `%lld` |
| `unsigned long long` | `ull`, `llu`, `ULL`, `LLU` | `42ULL` | `%llu` |
| `float` | `f` or `F` | `3.14f` | `%f`* |
| `double` | *(none)* | `3.14` | `%f` |
| `long double` | `l` or `L` | `3.14L` | `%Lf` |

\* **Important:** Although the type is `float`, arguments to `printf` undergo **default argument promotion**, so a `float` is automatically promoted to `double`. Therefore `%f` is the correct format specifier for both `float` and `double`.

## Integer Literal Suffix Summary

| Suffix | Type |
|---------|------|
| *(none)* | `int` (or larger if needed) |
| `u` | `unsigned int` |
| `l` | `long` |
| `ul` | `unsigned long` |
| `ll` | `long long` |
| `ull` | `unsigned long long` |

Suffixes are **case-insensitive** and may be written in either order:

```cpp
42UL
42LU
42ull
42LLU
```

are all valid.

## Floating-Point Literal Suffix Summary

| Suffix | Type |
|---------|------|
| *(none)* | `double` |
| `f` | `float` |
| `L` | `long double` |

Examples:

```cpp
3.14      // double
3.14f     // float
3.14L     // long double
```

## Common `printf` Integer Formats

| Type | Decimal | Unsigned | Hex | Octal |
|------|---------|----------|-----|--------|
| `signed char` | `%hhd` | — | — | — |
| `unsigned char` | — | `%hhu` | `%hhx` | `%hho` |
| `short` | `%hd` | — | — | — |
| `unsigned short` | — | `%hu` | `%hx` | `%ho` |
| `int` | `%d` | `%u` | `%x` | `%o` |
| `long` | `%ld` | `%lu` | `%lx` | `%lo` |
| `long long` | `%lld` | `%llu` | `%llx` | `%llo` |

## Common `printf` Floating Formats

| Format | Meaning |
|--------|---------|
| `%f` | Fixed-point notation |
| `%e` | Scientific notation |
| `%g` | Shorter of `%f` or `%e` |
| `%a` | Hexadecimal floating-point |
| `%Lf` | `long double` (fixed-point) |
| `%Le` | `long double` (scientific) |
| `%Lg` | `long double` (general) |
| `%La` | `long double` (hexadecimal) |


