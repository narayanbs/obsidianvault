
i have the following c program

#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main(void) {
	char *s = "hello whats'up how are you";
	int n = write(10, s, strlen(s));
	if(n <= 0) {
		return -1;
	}
	return 0;
}

if i compile and do ./test 10> out.txt,  I get the string inside out.txt.

how does the shell perform the miracle


Short answer: **file descriptors + `exec` = the “miracle.”**
Longer answer below 👇

---

### What your program is doing

In your C code:

```c
write(10, s, strlen(s));
```

You are explicitly writing to **file descriptor 10**. Normally:

* `0` → stdin
* `1` → stdout
* `2` → stderr

FD **10** is just… some number. Your program doesn’t open it. It just assumes it exists.

Yet somehow it works. That’s the shell’s magic ✨

---

### What the shell does when you run

```bash
./test 10> out.txt
```

The shell interprets `10> out.txt` **before your program starts**.

Conceptually, the shell does something like this:

1. **Open `out.txt`**

   ```c
   fd = open("out.txt", O_WRONLY | O_CREAT | O_TRUNC, 0666);
   ```

   Let’s say this returns fd `3`.

2. **Duplicate that fd to fd 10**

   ```c
   dup2(3, 10);
   ```

   Now:

   * FD 10 points to `out.txt`

3. **Close the original fd**

   ```c
   close(3);
   ```

4. **`execve("./test", ...)`**

   * The shell is replaced by your program
   * **All file descriptors remain open across `exec`**

---

### What your program sees

When `main()` starts:

* FD 10 already exists
* FD 10 already points to `out.txt`

So when you do:

```c
write(10, s, strlen(s));
```

The kernel just writes to the file.
Your program has **no idea** that redirection happened.

---

### Why this works at all

Key Unix ideas at play:

1. **File descriptors are inherited**

   * `fork()` → child gets copies
   * `exec()` → keeps them open

2. **Shell redirection is just fd manipulation**

   * `>` is syntax sugar for `open + dup2`

3. **Programs don’t know or care**

   * They just write to numbers

---

### You can prove it to yourself

Try this:

```bash
./test
```

Now FD 10 is *not* open → `write()` fails.

Or:

```bash
./test 10>&1
```

Now FD 10 points to stdout, so your string appears in the terminal.

---

### Mental model to keep forever

> **The shell rearranges file descriptors, then disappears.**

Your program walks into a room where the plumbing is already done.

No miracles. Just very elegant Unix design 😄

