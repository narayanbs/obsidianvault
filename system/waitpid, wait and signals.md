wait or signal handlers??? which should we use in parent process
---

## 1. Using `wait()` / `waitpid()` (Usually Preferred)

### How it works

After `fork()`, the parent explicitly calls:

```c
wait(NULL);
// or
waitpid(child_pid, &status, 0);
```

### Why it’s preferred

✅ **Simple and predictable**

- Control flow is explicit.
    
- Easy to reason about and debug.
    

✅ **No signal-related pitfalls**

- Signal handlers run asynchronously and can interrupt code at awkward times.
    
- You avoid reentrancy issues and race conditions.
    

✅ **Correct error handling**

- You get the child’s exit status reliably.
    
- You can wait for specific children (`waitpid`).
    

✅ **POSIX-recommended style**

- Most well-written Unix programs use `waitpid()` in a loop.
    

### Typical usage

```c
while (waitpid(-1, NULL, 0) > 0)
    ;
```

Best when:

- The parent already has a natural place to wait.
    
- You want deterministic child lifecycle management.
    
- You’re writing straightforward system code.
    

---

## 2. Using a `SIGCHLD` Handler

### How it works

The kernel sends `SIGCHLD` to the parent when a child exits. The handler calls `waitpid()`.

```c
void sigchld_handler(int sig) {
    while (waitpid(-1, NULL, WNOHANG) > 0)
        ;
}
```

### Why it’s _sometimes_ needed

✅ **Non-blocking parent**

- Parent can keep doing other work (e.g., a server).
    

✅ **Multiple children at unpredictable times**

- Common in long-running daemons or shells.
    

### Downsides

⚠ **More complex and error-prone**

- Signal handlers are asynchronous.
    
- Only async-signal-safe functions may be called.
    
- Easy to introduce subtle bugs.
    

⚠ **Race conditions**

- Child may exit before handler is installed.
    
- Must carefully block/unblock signals.
    

⚠ **Harder to maintain**

- Debugging signal-related bugs is notoriously difficult.
    

---

## 3. Special Case: Ignoring `SIGCHLD`

Some systems allow:

```c
signal(SIGCHLD, SIG_IGN);
```

This causes the kernel to reap children automatically **on Linux**, but:

- ⚠ **Not portable**
    
- ⚠ **Exit status is lost**
    
- ⚠ **Behavior differs across Unix variants**
    

Generally **not recommended** for portable code.

---

## 4. Summary: Which Should You Use?

|Situation|Preferred Method|
|---|---|
|Simple parent/child relationship|`wait()` / `waitpid()`|
|Need child exit status|`waitpid()`|
|Parent must not block (servers, shells)|`SIGCHLD` handler|
|Portable, maintainable code|`waitpid()`|
|Fire-and-forget children|`SIGCHLD` handler (carefully)|

---

Let’s walk through a **robust, race-free way to handle `SIGCHLD`**, the style used by shells and servers.

I’ll explain the **key ideas first**, then give you a **complete example**.

---

## Key Problems a Robust Design Must Solve

1. **Race condition**
    
    - A child might exit **before** the signal handler is installed.
        
2. **Multiple children**
    
    - One `SIGCHLD` can correspond to **many exited children**.
        
3. **Signal safety**
    
    - Only **async-signal-safe** functions can run inside the handler.
        
4. **No blocking**
    
    - The parent should continue doing useful work.
        

---

## Correct Design Pattern

### 1. Block `SIGCHLD` _before_ `fork()`

This prevents missing a signal.

### 2. Install the handler with `sigaction()`

(Not `signal()`, which is unreliable.)

### 3. In the handler:

- Call `waitpid()` in a loop
    
- Use `WNOHANG`
    
- Save and restore `errno`
    

### 4. Unblock `SIGCHLD`

After everything is set up.

---

## Robust Example Code

### SIGCHLD Handler

```c
#include <sys/wait.h>
#include <signal.h>
#include <errno.h>

void sigchld_handler(int sig)
{
    int saved_errno = errno;  // Preserve errno

    while (waitpid(-1, NULL, WNOHANG) > 0)
        ;

    errno = saved_errno;
}
```

✔ Uses only async-signal-safe functions  
✔ Handles multiple children  
✔ Prevents zombie buildup

---

### Installing the Handler (Race-Free)

```c
#include <signal.h>
#include <string.h>

void install_sigchld_handler(void)
{
    struct sigaction sa;
    sigemptyset(&sa.sa_mask);
    sa.sa_handler = sigchld_handler;
    sa.sa_flags = SA_RESTART;  // Restart interrupted syscalls

    sigaction(SIGCHLD, &sa, NULL);
}
```

---

### Forking Safely

```c
sigset_t mask, oldmask;

// Block SIGCHLD
sigemptyset(&mask);
sigaddset(&mask, SIGCHLD);
sigprocmask(SIG_BLOCK, &mask, &oldmask);

// Install handler
install_sigchld_handler();

pid_t pid = fork();

if (pid == 0) {
    // Child
    sigprocmask(SIG_SETMASK, &oldmask, NULL);
    _exit(0);
}

// Parent
sigprocmask(SIG_SETMASK, &oldmask, NULL);

// Parent continues working...
```

---

## Why This Works

✔ **No missed signals**  
SIGCHLD is blocked during setup.

✔ **No zombies**  
All dead children are reaped immediately.

✔ **No undefined behavior**  
Only safe functions in the handler.

✔ **Scales well**  
Handles thousands of children (common in shells and servers).

---

## How Real Programs Use This

- **Shells (`bash`, `zsh`)**
    
    - SIGCHLD handler reaps background jobs
        
    - Foreground jobs use `waitpid()` explicitly
        
- **Servers**
    
    - Handler reaps children
        
    - Main loop handles requests
        
- **Daemons**
    
    - Often combine SIGCHLD + event loop (`select`, `poll`, `epoll`)
        

---

## One Last Tip (Very Important)

> **Never put application logic in a signal handler.**

The handler should:

- reap children
    
- set a flag
    
- write to a pipe
    

Nothing more.