Sure 🙂 Here’s a **clean, practical list of C string-handling functions**, grouped by what they do. These come mostly from **`<string.h>`**, with a few from related headers that are commonly used alongside strings.

---

## 📦 Core C string functions (`<string.h>`)

### 🔹 Length

* `size_t strlen(const char *s)`

  * Returns length **excluding** the null terminator.

---

### 🔹 Copying

* `char *strcpy(char *dest, const char *src)`
* `char *strncpy(char *dest, const char *src, size_t n)`
* `char *strdup(const char *s)` *(POSIX, not ISO C)*

⚠️ `strncpy` is often misunderstood—does **not** guarantee null-termination.

---

### 🔹 Concatenation

* `char *strcat(char *dest, const char *src)`
* `char *strncat(char *dest, const char *src, size_t n)`

---

### 🔹 Comparison

* `int strcmp(const char *s1, const char *s2)`
* `int strncmp(const char *s1, const char *s2, size_t n)`
* `int strcasecmp(const char *s1, const char *s2)` *(POSIX, case-insensitive)*
* `int strncasecmp(const char *s1, const char *s2, size_t n)` *(POSIX)*

---

### 🔹 Searching

* `char *strchr(const char *s, int c)` – first occurrence of character
* `char *strrchr(const char *s, int c)` – last occurrence
* `char *strstr(const char *haystack, const char *needle)` – substring
* `char *strpbrk(const char *s, const char *accept)` – any of a set
* `size_t strspn(const char *s, const char *accept)`
* `size_t strcspn(const char *s, const char *reject)`

---

### 🔹 Tokenization

* `char *strtok(char *str, const char *delim)`
* `char *strtok_r(char *str, const char *delim, char **saveptr)` *(POSIX, thread-safe)*

---

### 🔹 Memory-based string helpers

(these work on raw memory, but are heavily used with strings)

* `void *memcpy(void *dest, const void *src, size_t n)`
* `void *memmove(void *dest, const void *src, size_t n)`
* `void *memset(void *s, int c, size_t n)`
* `int memcmp(const void *s1, const void *s2, size_t n)`
* `void *memchr(const void *s, int c, size_t n)`

---

### 🔹 Error strings

* `char *strerror(int errnum)`
* `int strerror_r(int errnum, char *buf, size_t buflen)` *(POSIX)*

---

## 📦 Safer / bounded alternatives (POSIX / BSD / GNU)

These are **not ISO C**, but common on Linux/macOS:

* `size_t strlcpy(char *dst, const char *src, size_t size)` *(BSD)*
* `size_t strlcat(char *dst, const char *src, size_t size)`
* `int asprintf(char **strp, const char *fmt, ...)`
* `int vasprintf(char **strp, const char *fmt, va_list ap)`

---

## 📦 Character classification (`<ctype.h>`)

(Operate on `char`, often used when parsing strings)

* `isalpha()`, `isdigit()`, `isalnum()`
* `isspace()`, `isupper()`, `islower()`
* `ispunct()`, `isxdigit()`
* `tolower()`, `toupper()`

---

## 📦 String ↔ number conversion (`<stdlib.h>`)

* `int atoi(const char *nptr)` *(unsafe, avoid)*
* `long atol(const char *nptr)`
* `double atof(const char *nptr)`
* `long strtol(const char *nptr, char **endptr, int base)`
* `long long strtoll(...)`
* `double strtod(...)`

✅ `strto*()` family is preferred.

---

## 🧠 Practical Advice (worth remembering)

* Prefer **`snprintf()`** over `sprintf()`
* Prefer **`strtok_r()`** over `strtok()` in real programs
* Avoid `strcpy`, `strcat`, and `atoi` in new code
* Always assume strings are **untrusted input**

---
