`scanf` is one of the most commonly used—but also most misunderstood—functions in C. It performs **formatted input parsing**, meaning it reads characters from `stdin` and interprets them according to a *format string*.

The key idea:

> `scanf` is not “reading variables” — it is **parsing a character stream using rules**.

---

# 1. Basic idea of how `scanf` works

Prototype:

```c
int scanf(const char *format, ...);
```

Internally, `scanf`:

1. Reads input character-by-character from standard input
2. Matches it against the format string
3. Converts matching parts into values
4. Stores them into provided variables
5. Stops when:

   * format is fully processed, OR
   * input doesn’t match expected pattern, OR
   * input ends

It returns:

* number of successfully assigned inputs
* `EOF` if input fails before any assignment

---

# 2. Format string parsing rules

A format string like:

```c
scanf("%d %f %s", &i, &f, str);
```

is processed left to right.

## Important rule: whitespace in format string

Any whitespace in format string means:

> “skip any amount of whitespace in input (including none)”

So:

```c
"%d %d"
```

will match:

```
10   20
```

or

```
10
20
```

---

# 3. How matching works (step-by-step idea)

Example:

```c
int a;
scanf("%d", &a);
```

Input:

```
123abc
```

Process:

* `%d` reads digits → `"123"`
* stops at `'a'` (non-digit)
* leaves `"abc"` in buffer

So:

```c
a = 123
```

---

# 4. Common format specifiers behavior

## `%d` (integer)

* Skips leading whitespace
* Reads optional sign
* Reads digits until non-digit

Input:

```
   -42xyz
```

Result: `-42`

---

## `%f`, `%lf`

* Floating-point parsing
* Accepts scientific notation

Input:

```
3.14e2abc
```

Result: `314.0`

---

## `%s` (VERY IMPORTANT)

```c
char str[10];
scanf("%s", str);
```

* Reads until **first whitespace**
* Does NOT protect against overflow

Input:

```
hello world
```

Result:

```
str = "hello"
```

`world` remains unread

---

## `%c` (tricky!)

```c
char c;
scanf("%c", &c);
```

* DOES NOT skip whitespace

Input:

```
A
```

→ reads `'A'`

Input:

```
   A
```

→ reads first space `' '`, NOT `'A'`

To skip whitespace:

```c
scanf(" %c", &c);  // leading space fixes it
```

---

# 5. Return value (very important in real code)

```c
int x, y;
int r = scanf("%d %d", &x, &y);
```

Possible values:

* `2` → both read successfully
* `1` → only first read
* `0` → no conversion happened
* `EOF` → input ended unexpectedly

---

# 6. Common tricky features

## (A) Width limit (prevents overflow)

```c
char str[5];
scanf("%4s", str);
```

* reads max 4 characters + `\0`
* safe(r) than `%s`

---

## (B) Suppression operator `*`

```c
int x;
scanf("%*d %d", &x);
```

Input:

```
10 20
```

* `10` is read but ignored
* `x = 20`

---

## (C) Scanset `[ ]` (very powerful)

```c
char str[100];
scanf("%[abc]", str);
```

Reads only characters `a`, `b`, `c`.

Input:

```
abacxyz
```

Result:

```
str = "abac"
```

Stops at `'x'`.

### Negated scanset:

```c
scanf("%[^\n]", str);
```

Reads until newline → commonly used for full line input.

---

# 7. Big pitfalls (very important)

## 1. Buffer overflow with `%s`

```c
char name[5];
scanf("%s", name); // DANGEROUS
```

Fix:

```c
scanf("%4s", name);
```

---

## 2. `%c` accidentally reading newline

```c
int x;
char c;

scanf("%d", &x);
scanf("%c", &c);  // reads '\n' from previous input
```

Fix:

```c
scanf(" %c", &c);
```

---

## 3. Mixing `scanf` and `fgets`

* `scanf` leaves leftover newline
* `fgets` reads it immediately

Example bug:

```c
scanf("%d", &x);
fgets(str, 100, stdin); // reads leftover '\n'
```

Fix:
consume newline or use consistent input method.

---

## 4. Input mismatch breaks parsing

```c
int x;
scanf("%d", &x);
```

Input:

```
abc
```

Result:

* `scanf` fails immediately
* input remains unread
* return value = 0

---

## 5. `scanf` is not line-based

People assume:

> “scanf reads a line”

Wrong.

It reads tokens, not lines.

---

# 8. Example combining everything

```c
int a;
float b;
char c;
char str[10];

scanf("%d %f %c %9s", &a, &b, &c, str);
```

Input:

```
42 3.5 Z hello_world
```

Result:

```
a = 42
b = 3.5
c = 'Z'
str = "hello_wo"   // truncated to 9 chars max (+ null)
```

---

# 9. Mental model to remember

Think of `scanf` as:

> A **cursor moving through input**, consuming characters only when they match the format rules.

Not:

* not “reading variables”
* not “reading lines”
* not “splitting strings automatically”

---

# If you want

I can also show:

* how `scanf` is implemented internally (simplified libc-style logic)
* comparison with `fgets + sscanf` (much safer pattern)
* or tricky exam questions based on `scanf` behavior
