`SA_RESTART` is a flag you can use when setting up signal handlers in Unix/Linux. Let’s go step by step.

---

## 1. What is `SA_RESTART`?

When a system call is **interrupted by a signal**, it can fail with `EINTR` (Interrupted system call).

- Many system calls (like `read()`, `write()`, `wait()`) are **blocking**.
    
- If a signal arrives while the system call is waiting, the default behavior is to **return immediately with an error `-1` and set `errno = EINTR`**.
- Note: If the signal is SIGINT,  default action is to terminate the process immediately w. 
    

`SA_RESTART` tells the system:

> “If a system call is interrupted by this signal, automatically **restart it** instead of failing.”

This makes signal handling smoother, because you don’t have to manually retry the system call every time.



---

### 1. **Signals in Linux**

Each signal can have one of three dispositions:

1. **Default action** – usually terminates the process or ignores the signal, depending on the signal.
    
2. **Ignore** – the process ignores the signal.
    
3. **Catch/handle** – the process installs a handler function to execute when the signal arrives.
    

---

### 2. **Signals and system calls**

When a process is blocked in a system call like `read()` or `write()`, a signal can **interrupt the system call**.

- If the signal is caught **and** the system call is **interruptible**, the call returns with `-1` and sets `errno = EINTR`.
    
- Example:
    

```c
ssize_t n = read(fd, buf, size);
if (n == -1 && errno == EINTR) {
    // read was interrupted by a signal
}
```

- Not all system calls are automatically restartable.
    

---

### 3. **SA_RESTART flag**

- When you set up a signal handler with `sigaction()`, you can specify the `SA_RESTART` flag.
    
- This tells the kernel:
    
    > If a system call is interrupted by this signal, **automatically restart the system call** instead of returning `-1` with `EINTR`.
    
- So yes, for `read()`, if it is blocked waiting for input and a signal is caught, with `SA_RESTART`, the kernel will **restart `read()` transparently** after the signal handler finishes.
    

---

### 4. **Important caveats**

1. **Not all syscalls are restartable.**
    
    - `read()`, `write()`, `wait()`, etc. are restartable.
        
    - `accept()`, `select()`, and some other syscalls may **not** be restarted automatically.
        
2. **Behavior depends on the type of file descriptor**
    
    - For terminal or network sockets, behavior can vary.
        
3. **If `SA_RESTART` is not set**
    
    - The call returns `-1` with `errno = EINTR` and you must handle it manually, often by looping:
        

```c
ssize_t n;
while ((n = read(fd, buf, size)) == -1 && errno == EINTR) {
    ; // retry
}
```



---

### **Without `SA_RESTART`**

1. `read()` is waiting for input (blocked).
    
2. A signal arrives, and you have a handler for it.
    
3. The kernel **pauses `read()`**, runs your signal handler.
    
4. After the handler finishes, **`read()` returns `-1`**, and `errno` is set to `EINTR`.
    
5. You, as the programmer, have to decide what to do next (e.g., retry `read()` manually).
    

---

### **With `SA_RESTART`**

1. `read()` is waiting for input.
    
2. A signal arrives, and you have a handler installed via `sigaction()` **with `SA_RESTART`**.
    
3. The kernel runs your signal handler.
    
4. After the handler finishes, the kernel **automatically restarts the `read()` call**, as if it was never interrupted.
    
5. Your program sees **no `-1` return; `read()` continues normally** until it completes.
    

---

### **Important points**

- **Not all syscalls are restartable**, even with `SA_RESTART`. But `read()`, `write()`, and many other basic I/O calls **are** restartable.
    
- If a syscall **cannot be restarted**, `SA_RESTART` has no effect, and `EINTR` is returned.
    
- Signal handlers themselves can do almost anything, but you should avoid blocking operations inside them.
    

---




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