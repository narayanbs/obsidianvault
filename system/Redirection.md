In Linux shell redirection, `<` and `<&` are **related but different**. The key difference is **what you are redirecting**: a _file_ vs a _file descriptor_.

---

## `<` — Redirect input from a **file**

`<` tells the shell to take **standard input (stdin)** from a file.

### Syntax

```bash
command < file
```

### Example

```bash
cat < input.txt
```

This is the same as:

```bash
cat input.txt
```

Here:

- `input.txt` is a **filename**
    
- stdin (file descriptor `0`) reads from that file
    

---

## `<&` — Redirect input from a **file descriptor**

`<&` tells the shell to take input from **another file descriptor**, not from a file.

### Syntax

```bash
command <&FD
```

### Example

```bash
command <&3
```

This means:

- stdin (`0`) now reads from **file descriptor 3**
    

⚠️ The file descriptor must already be open.

---

## Why `<&` is useful

File descriptors:

- `0` → stdin
    
- `1` → stdout
    
- `2` → stderr
    

### Example: redirect stdin from stdout

```bash
command <&1
```

### Example: close stdin

```bash
command <&-
```

---

## Side-by-side comparison

|Operator|Redirects from|Used for|
|---|---|---|
|`<`|A **file**|Normal input redirection|
|`<&`|A **file descriptor**|Advanced FD control|

---

## Simple mental model

- `<` → “read input from **this file**”
    
- `<&` → “read input from **this open stream**”
    

---


Here’s the **clear comparison of `>` vs `>&`** in Linux shell redirection, matching what you saw with `<` vs `<&`.

---

## `>` — Redirect output to a **file**

`>` sends **standard output (stdout)** to a file.

### Syntax

```bash
command > file
```

### Example

```bash
ls > out.txt
```

- stdout (`1`) goes to `out.txt`
    
- If the file exists → **overwritten**
    
- If not → **created**
    

(Use `>>` to append.)

---

## `>&` — Redirect output to a **file descriptor**

`>&` sends output to **another file descriptor**, not a file.

### Syntax

```bash
command >&FD
```

### Example

```bash
command >&2
```

- stdout (`1`) is redirected to **stderr (`2`)**
    

---

## Common real-world use: combine stdout and stderr

### Send stderr to stdout

```bash
command 2>&1
```

Meaning:

- `2>` → redirect stderr
    
- `&1` → to file descriptor 1 (stdout)
    

### Send both to a file

```bash
command > all.log 2>&1
```

Order matters:

1. stdout → `all.log`
    
2. stderr → where stdout now points
    

---

## Redirect both stdout and stderr (bash shortcut)

```bash
command &> all.log
```

Equivalent to:

```bash
command > all.log 2>&1
```

---

## Close an output stream

```bash
command 1>&-
```

Closes stdout.

---

## Side-by-side comparison

|Operator|Redirects to|Typical use|
|---|---|---|
|`>`|**File**|Normal output redirection|
|`>&`|**File descriptor**|Merge / duplicate streams|

---

## Big picture (input vs output)

|Input|Output|
|---|---|
|`< file`|`> file`|
|`<& FD`|`>& FD`|

---

## Easy memory trick

- `<` / `>` → **files**
    
- `<&` / `>&` → **file descriptors**
    
- `&` = “not a filename, a number”
    

---

If you want, I can also:

- explain why **order matters** in redirections
    
- show tricky interview questions
    
- compare `2>`, `&>`, `>|`, `exec` redirections
    

Just say the word 😊