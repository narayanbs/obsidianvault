## Given Function
```c

int fun(int n) {
  int x = 1, k;
  if (n == 1)
    return x;

  for (k = 1; k < n; ++k)
    x = x + fun(k) * fun(n - k);

  return x;
}
```
```
The **mathematical recurrence for the function’s return value**. That’s a different (and very important) skill 👍

Let’s derive it cleanly.

---

# 🔍 Given function

```c
int fun(int n) {
  int x = 1, k;
  if (n == 1)
    return x;

  for (k = 1; k < n; ++k)
    x = x + fun(k) * fun(n - k);

  return x;
}
```

---

# 🧠 Step 1: Define what the function returns

Let:
$$
f(n) = \text{value returned by fun(n)}
$$

---

# 🧩 Step 2: Base case

From code:

```c
if (n == 1)
  return 1;
```

So:
$$
f(1) = 1
$$

---

# 🔁 Step 3: Translate the loop

The key line is:

```c
x = x + fun(k) * fun(n - k);
```

* Initially: `x = 1`
* Then for each (k = 1) to (n-1), we add:
  $$
  f(k) \cdot f(n-k)
  $$

---

# ✨ Step 4: Build the recurrence

Start with the initial value:

$$
f(n) = 1 + \sum_{k=1}^{n-1} f(k),f(n-k)
$$

---

# ✅ Final answer

$$
\boxed{
f(n) = 1 + \sum_{k=1}^{n-1} f(k),f(n-k)
}
\quad \text{with } f(1) = 1
$$

---

# 💡 Why the “+1” is there

Because:

```c
int x = 1;
```

That initial value directly contributes the **+1** in the recurrence.

If instead it was:

```c
int x = 0;
```

Then the recurrence would be:
$$
f(n) = \sum_{k=1}^{n-1} f(k),f(n-k)
$$

---

# 🔥 Key takeaway

When converting code → recurrence for output:

1. **Base case → base value**
2. **Initial variable value → constant term**
3. **Loop → summation**
4. **Recursive calls → function terms**

---

If you want, I can give you a few similar functions and let you practice deriving the recurrence (this is exactly the kind of thing exams love).




=========================================================================================

Perfect—that’s exactly the right way to get good at this. I’ll give you **carefully chosen examples** that build intuition step by step. Try each one yourself first, then check the answer.

---

# 🧪 Example 1 (warm-up)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 0;
    for (int i = 1; i < n; i++)
        x += f(i);
    return x;
}
```

### 👉 Try to derive:

* What is ( f(1) )?
* What does the loop add?

---

### ✅ Answer

* Base case:
  $$
  f(1) = 1
  $$

* Loop adds ( f(1) + f(2) + \dots + f(n-1) )

So:
$$
\boxed{
f(n) = \sum_{i=1}^{n-1} f(i)
}
$$

---

# 🧪 Example 2 (constant + sum)

```c
int f(int n) {
    if (n == 1) return 2;
    int x = 3;
    for (int i = 1; i < n; i++)
        x += f(i);
    return x;
}
```

---

### ✅ Answer

* Base:
  $$
  f(1) = 2
  $$

* Start value = 3

* Loop adds sum

$$
\boxed{
f(n) = 3 + \sum_{i=1}^{n-1} f(i)
}
$$

---

# 🧪 Example 3 (product inside loop)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 0;
    for (int i = 1; i < n; i++)
        x += f(i) * i;
    return x;
}
```

---

### ✅ Answer

$$
f(1) = 1
$$

$$
\boxed{
f(n) = \sum_{i=1}^{n-1} i \cdot f(i)
}
$$

---

# 🧪 Example 4 (two recursive calls, no product)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 0;
    for (int i = 1; i < n; i++)
        x += f(i) + f(n-i);
    return x;
}
```

---

### ✅ Answer

Split the sum:

$$
f(n) = \sum_{i=1}^{n-1} \big(f(i) + f(n-i)\big)
$$

$$
= \sum_{i=1}^{n-1} f(i) + \sum_{i=1}^{n-1} f(n-i)
$$

Symmetry:

$$
\boxed{
f(n) = 2 \sum_{i=1}^{n-1} f(i)
}
$$

---

# 🧪 Example 5 (very important pattern — like yours)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 0;
    for (int i = 1; i < n; i++)
        x += f(i) * f(n-i);
    return x;
}
```

---

### ✅ Answer

$$
\boxed{
f(n) = \sum_{i=1}^{n-1} f(i),f(n-i)
}
$$

---

# 🧪 Example 6 (your exact pattern + constant)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 5;
    for (int i = 1; i < n; i++)
        x += f(i) * f(n-i);
    return x;
}
```

---

### ✅ Answer

$$
\boxed{
f(n) = 5 + \sum_{i=1}^{n-1} f(i),f(n-i)
}
$$

---

# 🧠 Pattern recognition cheat sheet

| Code pattern       | Recurrence         |
| ------------------ | ------------------ |
| `x = 0; x += f(i)` | ( \sum f(i) )      |
| `x = C; x += ...`  | ( C + \sum ... )   |
| `f(i) * f(n-i)`    | product term       |
| `f(i) + f(n-i)`    | splits into 2 sums |
| loop `1 → n-1`     | summation limits   |

---

# 🔥 One for you to try (important)

```c
int f(int n) {
    if (n == 1) return 1;
    int x = 2;
    for (int i = 1; i < n; i++)
        x += f(i) * f(i);
    return x;
}
```

👉 Try to derive the recurrence.

---

If you want, I can:

* Check your answer
* Give harder exam-level questions
* Or show tricks to simplify these recurrences quickly
