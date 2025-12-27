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

|Specifier|Meaning|Example Output|
|---|---|---|
|`%d` / `%i`|signed integer|`10`, `-3`|
|`%u`|unsigned integer|`20`|
|`%f`|floating-point|`3.14`|
|`%c`|single character|`A`|
|`%s`|string|`Hello`|
|`%ld`|long int|`2000000000`|
|`%lld`|long long int|`9000000000000`|
|`%lf`|double|`3.141500`|
|`%p`|memory address|`0x7ffeefbff45c`|
|`%%`|prints `%` itself|`%`|

---

# ✅ **2. Integer Specifiers**

### **Decimal**

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

|Specifier|Meaning|
|---|---|
|`%f`|float/double (fixed-point)|
|`%e` / `%E`|scientific notation|
|`%g` / `%G`|compact form (uses `%f` or `%e`)|

### Examples:

```c
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

```c
printf("%.2f", 3.14159);   // 3.14
```

### **Precision for Strings**

```c
printf("%.5s", "HelloWorld");   // Hello
```

---

# ✅ **6. Flags**

|Flag|Meaning|
|---|---|
|`-`|left-justify|
|`+`|force sign|
|`0`|pad with zeros|
||space before positive numbers|
|`#`|special form (hex `0x`, octal `0`)|

### Examples:

```c
printf("%+d", 10);    // +10
printf("%05d", 42);   // 00042
printf("%-5d", 42);   // 42___
printf("%#x", 255);   // 0xff
```

---

# ✅ **7. Length Modifiers**

|Modifier|Used For|Example|
|---|---|---|
|`h`|short|`%hd`|
|`l`|long|`%ld`, `%lf`|
|`ll`|long long|`%lld`|
|`L`|long double|`%Lf`|

---

# ⚡ **8. Complete Examples**

### Example 1: Mixed Output

```c
int a = 10;
float b = 3.14;
char c = 'X';

printf("a=%d, b=%.2f, c=%c", a, b, c);
```

**Output:**

```
a=10, b=3.14, c=X
```

---

### Example 2: Formatting Table

```c
printf("%-10s | %5d | %8.2f\n", "Item", 12, 5.678);
```

**Output:**

```
Item       |    12 |    5.68
```

---

# 🎯 Summary Cheat-Sheet

```
%d  integer
%u  unsigned int
%f  float/double
%c  char
%s  string
%x/%X  hex
%o  octal
%p  pointer
%ld/%lld  long / long long
```

---

If you want, I can also generate:

📌 a printable cheat-sheet  
📌 exercises with solutions  
📌 a visual diagram of the `printf` format structure

Just tell me!