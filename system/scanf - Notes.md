🧠 1. What is `scanf()`?

`scanf()` is a **standard input function** in C used to **read data** from the **standard input stream** (`stdin`, usually the keyboard).

```c
scanf("format_specifier", &variable1, &variable2, ...);
```

## 🧩 Common Format Specifiers

| Data Type           | Format Specifier | Example              |
| ------------------- | ---------------- | -------------------- |
| `int`               | `%d`             | `scanf("%d", &x);`   |
| `float`             | `%f`             | `scanf("%f", &f);`   |
| `double`            | `%lf`            | `scanf("%lf", &d);`  |
| `char`              | `%c`             | `scanf("%c", &ch);`  |
| `string` (`char[]`) | `%s`             | `scanf("%s", str);`  |
| `unsigned int`      | `%u`             | `scanf("%u", &num);` |
| `hexadecimal int`   | `%x` or `%X`     | `scanf("%x", &num);` |

`scanf()` reads input from the standard input (usually the keyboard) and stores it in variables according to the *format specifiers* you provide.

Example:

```c
int a;
scanf("%d", &a);
```
This reads an integer from input.

```c
#include <stdio.h>

int main() {
    int a;
    float b;
    printf("Enter an integer and a float: ");
    scanf("%d %f", &a, &b);
    printf("You entered: %d and %.2f\n", a, b);
    return 0;
}
```
This reads multiple values

In scanf whitespace characters in the format string — such as space ' ', tab '\t', and newline '\n' — all mean “skip any amount of whitespace (including newlines) in the input.”

So:

```c
scanf("%d\n%d", &a, &b);
```

behaves the same as:

```c
scanf("%d %d", &a, &b);
```

Both will skip over any spaces, tabs, or newlines between numbers.

> ⚠️ **Important note:** The `\n` at the *end* of a format string (e.g. `scanf("%d\n", &a);`) can cause `scanf` to **wait indefinitely** for a non-whitespace character — because it expects *something* after the whitespace.

Example of a problematic case:

```c
int x;
scanf("%d\n", &x); // BAD: scanf will wait for another non-whitespace input
```
## 🔤 Character Format Specifier (`%c`)

When you use `%c`, `scanf` reads **exactly one character** — including spaces, tabs, and newlines.

Example:

```c
char ch;
scanf("%c", &ch);
```

If you type:

```
A⏎
```

`ch` will get `'A'` and the newline `'\n'` will remain in the input buffer.

This can cause confusion if you read another `%c` or `%s` afterward — because the leftover `'\n'` will be read next time.
## ⚙️ Example: Newline remaining in the input buffer

```c
#include <stdio.h>

int main() {
    int num;
    char ch;

    printf("Enter a number: ");
    scanf("%d", &num);        // reads the number, leaves '\n' in buffer

    printf("Enter a character: ");
    scanf("%c", &ch);         // tries to read next character (gets '\n'!)

    printf("num = %d, ch = %c\n", num, ch);

    return 0;
}
```

---

### 🧩 Input:

```
5⏎
A⏎
```

Let’s see what actually happens step-by-step:

| Step | Input Stream | What Happens                                                                                           |
| ---- | ------------ | ------------------------------------------------------------------------------------------------------ |
| 1    | `5⏎A⏎`       | You type `5` and press Enter. So the input stream now has the characters `'5'`, `'\n'`, `'A'`, `'\n'`. |
| 2    | `5⏎A⏎`       | `scanf("%d", &num);` reads the `5` and stops at the newline (`'\n'`), **leaving it in the buffer**.    |
| 3    | `⏎A⏎`        | Now `scanf("%c", &ch);` runs — it reads **the very next character**, which is the leftover `'\n'`.     |
| 4    | `A⏎`         | `ch` now contains `'\n'`, *not* `'A'`. So you’ll see no visible output for `ch`.                       |

---

### 🧾 Output:

```
Enter a number: 5
Enter a character: num = 5, ch =
```

Notice `ch` appears blank — it actually contains a newline.

---

## ✅ Fix: Skip the leftover newline

The trick is to put a space before `%c` in the format string:

```c
scanf(" %c", &ch);
```

That space tells `scanf()` to skip any whitespace (spaces, tabs, or newlines) before reading the next character.

Revised program:

```c
#include <stdio.h>

int main() {
    int num;
    char ch;

    printf("Enter a number: ");
    scanf("%d", &num);

    printf("Enter a character: ");
    scanf(" %c", &ch);   // note the leading space!

    printf("num = %d, ch = %c\n", num, ch);

    return 0;
}
```

### Input:

```
5⏎
A⏎
```

### Output:

```
Enter a number: 5
Enter a character: A
num = 5, ch = A
```

Now it works perfectly! ✅

---

## 💡 6. Advanced Usage

### 6.1 Field Width

You can limit how many characters `scanf()` reads.

Example:

```c
char name[10];
scanf("%9s", name); // reads at most 9 chars + '\0'
```

### 6.2 Suppressing Input

Use `*` to read but ignore input.

```c
int day, month, year;
scanf("%d/%d/%d", &day, &month, &year);
```

Or:

```c
scanf("%*d-%d", &num); // skips the first number
```

### 6.3 Scanset (`[]`)

To define acceptable characters:

```c
char str[50];
scanf("%[A-Za-z]", str); // reads only letters
```

---

## 🚫 7. Common Pitfalls

| Mistake                   | Problem                             | Solution                       |
| ------------------------- | ----------------------------------- | ------------------------------ |
| Forgetting `&`            | Program crash or wrong value        | Always use `&` for non-strings |
| Using `%s` for sentences  | Stops at space                      | Use `%[^\n]` or `fgets()`      |
| Not checking return value | Undefined behavior on invalid input | Check `scanf()` return value   |
| Buffer overflow           | Writing beyond array size           | Limit input width (e.g. `%9s`) |

---

## 🧱 8. Comparison: `scanf()` vs `fgets()`

| Feature                    | `scanf()` | `fgets()` |
| -------------------------- | --------- | --------- |
| Reads until whitespace     | ✅         | ❌         |
| Reads entire line (spaces) | ❌         | ✅         |
| Easier for multiple values | ✅         | ❌         |
| Safer (less overflow risk) | ❌         | ✅         |

> 👉 Best practice: use `fgets()` + `sscanf()` for safer input parsing.

Example:

```c
char line[100];
int a; float b;
fgets(line, sizeof(line), stdin);
sscanf(line, "%d %f", &a, &b);
```

---

## 🧾 9. Full Example Program

```c
#include <stdio.h>

int main() {
    int age;
    float height;
    char name[30];

    printf("Enter your name: ");
    scanf("%29s", name);

    printf("Enter your age and height: ");
    if (scanf("%d %f", &age, &height) == 2) {
        printf("Name: %s\nAge: %d\nHeight: %.2f\n", name, age, height);
    } else {
        printf("Invalid input!\n");
    }

    return 0;
}
```

---

## 🧭 10. Summary Table

| Task               | Example                 | Notes                              |
| ------------------ | ----------------------- | ---------------------------------- |
| Read int           | `scanf("%d", &x);`      | Must use `&`                       |
| Read float         | `scanf("%f", &f);`      | Must use `&`                       |
| Read char          | `scanf(" %c", &ch);`    | Skip spaces with space before `%c` |
| Read string        | `scanf("%s", str);`     | Stops at whitespace                |
| Read line          | `scanf("%[^\n]", str);` | Reads until newline                |
| Limit input length | `scanf("%9s", str);`    | Prevents overflow                  |

---

Would you like me to include **interactive exercises** (with example input/output) so you can practice `scanf()` concepts step-by-step?


Excellent question 👏 — this is one of the trickiest but most interesting parts of how **`scanf()`** works!

Let’s break down exactly **why `%[^\n]` works for reading input with spaces**, and what’s happening behind the scenes.

---

## 🧩 1. `%[...]` — The "Scanset" Format Specifier

In `scanf()`, the format specifier **`%[...]`** (called a *scanset*) means:

> “Read and store characters that belong to this set until a character **not in the set** is found.”

So:

```c
scanf("%[abc]", str);
```

means:
👉 Read characters as long as they are `'a'`, `'b'`, or `'c'`.

If the user enters:

```
abbcabz
```

Then `str` will contain `"abbcab"` — scanning stops when `'z'` (not in `[abc]`) is found.

---

## 💡 2. `%[^\n]` — The “Read Until Newline” Trick

Let’s read this carefully:

* `^` (caret) inside the brackets means **negation**.
* `\n` represents the **newline character** (the Enter key).

So:

```c
scanf("%[^\n]", str);
```

means:

> "Read **everything except newline** (`\n`) and stop when you see a newline."

---

## ✨ 3. Why It Reads Spaces (Unlike `%s`)

The normal `%s` format specifier:

```c
scanf("%s", str);
```

stops reading input when it sees *any whitespace* — space, tab, or newline.

But `%[^\n]` only stops at **newline**, not at spaces or tabs.

So if you type:

```
Hello world this is C
```

Then:

* `%s` → only reads `"Hello"` (stops at the space)
* `%[^\n]` → reads `"Hello world this is C"` (stops only when you press Enter)

That’s why `%[^\n]` is perfect for reading **sentences or full lines**.

---

## 🧮 4. Example in Action

```c
#include <stdio.h>

int main() {
    char line[100];
    printf("Enter a sentence: ");
    scanf("%[^\n]", line);
    printf("You entered: %s\n", line);
    return 0;
}
```

**Input:**

```
I love programming in C
```

**Output:**

```
You entered: I love programming in C
```

---

## ⚠️ 5. Important Notes

### 🧱 1. No automatic skipping of newlines

Unlike `%s`, the `%[^\n]` format **doesn’t skip leading newlines**.
So if there’s a leftover newline in the input buffer (from a previous `scanf`), it will cause problems.

✅ To fix it, you can add a space before `%`:

```c
scanf(" %[^\n]", line);
```

That space tells `scanf()` to skip any whitespace (including leftover newlines).

---

### 🧱 2. Always limit the input size

To prevent overflow:

```c
scanf(" %99[^\n]", line);  // reads up to 99 chars + '\0'
```

---

## 🧠 6. Summary

| Specifier | Stops Reading At | Reads Spaces? | Notes                                  |
| --------- | ---------------- | ------------- | -------------------------------------- |
| `%s`      | Any whitespace   | ❌             | Good for single words                  |
| `%c`      | No stopping      | ✅             | Reads raw characters (even whitespace) |
| `%[^\n]`  | Newline (`\n`)   | ✅             | Reads full line (including spaces)     |

---

## 🧭 7. Practical Tip

While `%[^\n]` works great for quick programs, for safer and more flexible input, prefer:

```c
fgets(line, sizeof(line), stdin);
```

It does the same thing (reads spaces, stops at newline) and **avoids buffer overflows**.

---

Would you like me to show the difference between `%[^\n]` and `fgets()` side by side with real input/output examples?

----------------------------

Let’s break it down carefully:

Your code is:

```c
int c;
scanf("%*d %d", &c);
printf("%d\n", c);
```

And your **input** is:

```
Hello 400 500
```

---

### Step 1: Understand the `scanf` format string

`%*d`

- This means: “read an **integer**, but do **not** assign it to a variable” (the `*` suppresses assignment).
    

`%d`

- This means: “read an **integer** and store it into the variable (in this case, `c`).”
    

So `scanf` is expecting **two integers**, separated by whitespace or other valid separators.

---

### Step 2: What happens with the input

Input: `"Hello 400 500"`

- `scanf` starts scanning the input.
    
- It first expects an integer (`%*d`), but the first thing in the input is `"Hello"`, which **is not a number**.
    

`scanf` will stop immediately at the first non-numeric input when trying to match `%d`.

Thus:

- `%*d` **fails** to match anything,
    
- The function **stops reading**,
    
- It **does not proceed to** the next `%d`,
    
- The variable `c` is **never assigned**.
    

---

### Step 3: Result

- `scanf` returns `0` because **no input items were successfully matched**.
    
- The value of `c` is **indeterminate (garbage)** because it was never written to.
    

---

✅ **Final Answer:**

`c` has an **undefined (garbage)** value — whatever random bits happened to be in that memory location.

If you want to check it safely, you could initialize it:

```c
int c = 0;
scanf("%*d %d", &c);
printf("%d\n", c);
```

Then if the first read fails, `c` would still print as `0`.