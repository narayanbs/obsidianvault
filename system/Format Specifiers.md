Below is a clear, practical tutorial on **format specifiers in C's `printf()`**, with examples and tips.

---

# 📘 **Format Specifiers in `printf()` – Complete Tutorial**

`printf()` is used to print data to the screen.  
Format specifiers tell `printf()` **what type of data** you want to print.

A format specifier always begins with `%` and is optionally followed by:

```
%[flags][width][.precision][length]specifier
```

But in simple usage, you mainly use:

```
%specifier
```

---

# ✅ **1. Basic Format Specifiers**


| Format Specifier | Data Type          | Description / Example Output                                                                                          |
| :--------------- | :----------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `%d` or `%i`     | `int`              | Signed decimal integer (e.g., `42`, `-10`)                                                                            |
| `%u`             | `unsigned int`     | Unsigned decimal integer (e.g., `42`)                                                                                 |
| `%f`             | `float`            | Decimal floating-point (e.g., `3.141593`)                                                                             |
| `%lf`            | `double`           | Used in `scanf` for double precision (Note: `printf` accepts `%f` for both float and double due to default promotion) |
| `%e` or `%E`     | `float` / `double` | Scientific notation (e.g., `3.141593e+00`)                                                                            |
| `%c`             | `char`             | Single character (e.g., `A`)                                                                                          |
| `%s`             | `char *`           | String of characters (e.g., `Hello, World!`)                                                                          |
| `%p`             | `void *`           | Pointer address in hexadecimal (e.g., `0x7ffee6b840bc`)                                                               |
| `%x` or `%X`     | `unsigned int`     | Hexadecimal integer (lowercase `a-f` or uppercase `A-F`)                                                              |
| `%o`             | `unsigned int`     | Octal integer (e.g., `52`)                                                                                            |
| `%%`             | *None*             | Prints a literal percent sign (`%`)                                                                                   |

### Common Modifiers
You can combine these specifiers with flags to control width, precision, and alignment:
* `%.2f`: Rounds a floating-point number to 2 decimal places (e.g., `3.14`).
* `%5d`: Pads the integer with spaces to ensure a minimum width of 5 characters.
* `%-5d`: Left-aligns the integer within a width of 5 characters.

---

# ✅ **2. Integer Specifiers**

### **Signed**

```c
printf("%d", 25);      // 25
```

### **Unsigned**

```c
printf("%u", 25u);     // 25
```

### **Octal**

```c
printf("%o", 25);      // 31
```

### **Hexadecimal**

```c
printf("%x", 255);     // ff
printf("%X", 255);     // FF
```

### **Long / Long Long**

```c
printf("%ld", 1234567890L);
printf("%lld", 123456789012345LL);
```

---

# ✅ **3. Floating-Point Specifiers**

| Specifier   | Meaning                          |
| ----------- | -------------------------------- |
| `%a / %A`   | Hexadecimal floating point       |
| `%f`        | float/double (fixed-point)       |
| `%e` / `%E` | scientific notation              |
| `%g` / `%G` | compact form (uses `%f` or `%e`) |

### Examples:

```c
printf("%a", 1216.0);  // 0x1.3p10
printf("%A", 1216.0);  // 0x1.3P10
printf("%f", 3.14159);     // 3.141590
printf("%0.2f", 3.14159);  // 3.14
printf("%e", 3.14159);     // 3.141590e+00
printf("%g", 3.14159);     // 3.14159
```

---

# ✅ **4. Character and String**

```c
printf("%c", 'A');         // A
printf("%s", "Hello");     // Hello
```

---

# ✅ **5. Width and Precision**

### **Minimum Width**

```c
printf("%10d", 25);   // "        25"
```

### **Precision for Floating Numbers**

A width of `10.2` represents minimum 10 digits including the decimal point, with 2 digits after the decimal point. 

```c
printf("%10.2f", 3.14159);   // 3.14
```

### **Precision for Strings**

```c
printf("%.5s", "HelloWorld");   // Hello
```

---

# ✅ **6. Flags**

| Flag | Meaning                            |
| ---- | ---------------------------------- |
| `-`  | left-justify                       |
| `+`  | force sign                         |
| `0`  | pad with zeros                     |
|      | space before positive numbers      |
| `#`  | special form (hex `0x`, octal `0`) |

### Examples:

```c
printf("%+d", 10);    // +10
printf("%05d", 42);   // 00042
printf("%-5d", 42);   // 42___
printf("%#x", 255);   // 0xff
```

---

# ✅ **7. Length Modifiers**

| Modifier | Used For    | Example            |
| -------- | ----------- | ------------------ |
| `hh`     | char        | `%hhd`             |
| `h`      | short       | `%hd`              |
| `l`      | long        | `%ld`, `%lu`,`%lf` |
| `ll`     | long long   | `%lld`, `%llu`     |
| `L`      | long double | `%Lf`              |

---

# Examples

Sure! `printf` has a lot of format specifiers beyond the common `%d` and `%s`. Here are some interesting examples in C.

### Basic integer formatting

```c
#include <stdio.h>

int main() {
    int n = 42;

    printf("%d\n", n);      // 42
    printf("%5d\n", n);     // "   42" (width 5)
    printf("%-5d|\n", n);   // "42   |" (left aligned)
    printf("%05d\n", n);    // "00042" (zero padded)
}
```

---

### Hexadecimal, octal, and binary (non-standard)

```c
int x = 255;

printf("%x\n", x);   // ff
printf("%X\n", x);   // FF
printf("%#x\n", x);  // 0xff
printf("%o\n", x);   // 377
printf("%#o\n", x);  // 0377
```

> Note: Standard `printf` does **not** support `%b` for binary. Some compilers or libraries add it as an extension.

---

### Floating point precision

```c
double pi = 3.141592653589793;

printf("%f\n", pi);      // 3.141593
printf("%.2f\n", pi);    // 3.14
printf("%10.2f\n", pi);  // "      3.14"
printf("%-10.2f|\n", pi);// "3.14      |"
```

---

### Scientific notation

```c
double x = 123456.789;

printf("%e\n", x);   // 1.234568e+05
printf("%E\n", x);   // 1.234568E+05
```

---

### `%g` — choose the shorter representation

```c
double a = 123.456;
double b = 0.00000123;

printf("%g\n", a);   // 123.456
printf("%g\n", b);   // 1.23e-06
```


### `%a or %A` -- Hexadecimal representation of float

```c
#include <stdio.h>

int main(void)
{
    double d = 3.1514234;

    printf("%a\n", d);
    printf("%A\n", d);
}

output:
0x1.9360d4d85c4c8p+1
0X1.9360D4D85C4C8P+1
```

### Print 64-bit IEEE-754 representation

```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>
#include <inttypes.h>

int main(void)
{
    double d = 3.1514234;

    uint64_t bits;
    memcpy(&bits, &d, sizeof bits);

    printf("0x%016" PRIx64 "\n", bits);
}

output: 
0x4009360d4d85c4c8
```
***Note: PRIx64 is implementation defined. could be `lx` or `llx`*** 

---

### Character and string

```c
char c = 'A';
char *s = "Hello";

printf("%c\n", c);
printf("%s\n", s);

printf("%.3s\n", s);    // Hel
printf("%10s\n", s);    // "     Hello"
printf("%-10s|\n", s);  // "Hello     |"
```

---

### Pointer addresses

```c
int x = 42;

printf("%p\n", (void *)&x);
```

Example output:

```
0x7ffc9f8c12b4
```

---

### Printing a literal `%`

```c
printf("Progress: 75%% complete\n");
```

Output:

```
Progress: 75% complete
```

---

### Dynamic width and precision

```c
double pi = 3.14159265;

printf("%*.*f\n", 8, 3, pi);
```

Equivalent to:

```c
printf("%8.3f\n", pi);
```

Output:

```
   3.142
```

---

### Thousands grouping (locale-dependent)

```c
#include <locale.h>

setlocale(LC_ALL, "");

printf("%'d\n", 123456789);
```

Possible output (depends on locale):

```
123,456,789
```

or

```
123 456 789
```

---

### Alternative form (`#`)

```c
printf("%#x\n", 255);      // 0xff
printf("%#o\n", 255);      // 0377
printf("%#.0f\n", 5.0);    // 5.
```

The `#` flag forces prefixes (`0x`, `0`) or ensures a decimal point is present for floating-point values.

---

### Always show sign

```c
printf("%+d\n", 42);   // +42
printf("%+d\n", -42);  // -42
```

---

### Space for positive numbers

```c
printf("% d\n", 42);   // " 42"
printf("% d\n", -42);  // "-42"
```

Useful for aligning signed numbers.

---

### Length modifiers

```c
char c = 'a';
short s = 123;
long l = 123456789L;
long long ll = 1234567890123LL;

printf("%hhd\n", c);
printf("%hd\n", s);
printf("%ld\n", l);
printf("%lld\n", ll);
```

Unsigned versions:

```c
unsigned int u = 42;
size_t sz = 100;
unsigned long l = 123456789L;
unsigned long long ll = 1234567890123LL;

printf("%u\n", u);
printf("%zu\n", sz);
printf("%lu\n", l);
printf("%llu\n", ll);
```

---

### `%n` — number of characters printed so far

```c
int count;

printf("Hello%n World\n", &count);

printf("%d\n", count);
```

Output:

```
Hello World
5
```

`%n` does **not** print anything. Instead, it stores the number of characters printed so far into the provided integer pointer.

> This specifier is powerful but also security-sensitive. If untrusted input controls the format string, `%n` can be abused, which is why some libraries disable or restrict it.

---


