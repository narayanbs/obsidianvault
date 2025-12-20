`SA_RESTART` is a flag you can use when setting up signal handlers in Unix/Linux. Let’s go step by step.

---

## 1. What is `SA_RESTART`?

When a system call is **interrupted by a signal**, it can fail with `EINTR` (Interrupted system call).

- Many system calls (like `read()`, `write()`, `wait()`) are **blocking**.
    
- If a signal arrives while the system call is waiting, the default behavior is to **return immediately with an error `-1` and set `errno = EINTR`**.
    

`SA_RESTART` tells the system:

> “If a system call is interrupted by this signal, automatically **restart it** instead of failing.”

This makes signal handling smoother, because you don’t have to manually retry the system call every time.

---

## 2. How to use `SA_RESTART`

You set it with `sigaction`:

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    printf("Caught signal %d\n", sig);
}

int main() {
    struct sigaction sa;
    sa.sa_handler = handler;
    sa.sa_flags = SA_RESTART; // automatically restart interrupted syscalls
    sigemptyset(&sa.sa_mask);
    sigaction(SIGINT, &sa, NULL);

    char buf[10];
    printf("Enter something: ");
    ssize_t n = read(STDIN_FILENO, buf, sizeof(buf)); // might be interrupted
    if (n < 0) {
        perror("read");
    } else {
        printf("Read %zd bytes\n", n);
    }
}
```

---

## 3. What happens without `SA_RESTART`

- Pressing **Ctrl+C** sends `SIGINT`.
    
- If `SA_RESTART` is **not set**, `read()` can return `-1` with `errno = EINTR`.
    
- You’d have to manually retry the `read()` in a loop.
    

With `SA_RESTART`:

- The `read()` call **resumes automatically** after the signal handler finishes.
    
- The user doesn’t see an error; the program continues smoothly.
    

---

## 4. Common use cases

- **Input/output**: `read()`, `write()`, `recv()`
    
- **Process waiting**: `wait()`, `waitpid()`
    
- Anywhere a **blocking system call** might be interrupted by a signal
    

> ⚠️ Note: Not all syscalls are automatically restartable. Some, like `select()`, `sleep()`, or `poll()`, may still return `EINTR` even with `SA_RESTART`.

---

If you want, I can make a **small demo comparing `SA_RESTART` vs no `SA_RESTART`**, so you can see `read()` get interrupted vs automatically restarted.

Do you want me to do that?