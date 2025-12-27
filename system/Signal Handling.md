
## 1️⃣ What is a signal mask?

Every thread has a **signal mask**: a set of signals that are **blocked** (i.e., delivery is deferred).

If a blocked signal is sent:

- it becomes _pending_
    
- it is delivered only after being unblocked
    

---

## 2️⃣ Step-by-step explanation

### Declare variables

```c
sigset_t set, old;
```

- `set` → the signals you want to block
    
- `old` → the previous signal mask (saved for restoration)
    

---

### Initialize the set to empty

```c
sigemptyset(&set);
```

- Clears all signals from `set`
    
- Now `set` blocks nothing
    

---

### Add SIGINT to the set

```c
sigaddset(&set, SIGINT);
```

- `set` now contains **only SIGINT**
    

---

### Block SIGINT and save the old mask

```c
sigprocmask(SIG_BLOCK, &set, &old);
```

This does two things **atomically**:

1. Adds `SIGINT` to the current signal mask
    
2. Saves the _previous_ signal mask in `old`
    

From this point on:

- SIGINT (Ctrl-C) **will not be delivered**
    
- If SIGINT occurs, it becomes _pending_
    

---

### Critical section

```c
/* critical section */
```

Code here:

- cannot be interrupted by SIGINT
    
- is protected from race conditions involving SIGINT handlers
    

---

### Restore the old signal mask

```c
sigprocmask(SIG_SETMASK, &old, NULL);
```

- Restores the signal mask exactly as it was before
    
- If a SIGINT arrived while blocked:
    
    - it is delivered **now**
        
    - the handler runs immediately after unblocking
        

---

## 3️⃣ Why this is important

Without this:

```c
shared_data++;
```

A signal handler could interrupt halfway and corrupt state.

With this:

- Either the handler runs **before**
    
- Or **after**
    
- But never _during_ the critical section
    

---

## 4️⃣ Why not just disable the handler?

Because:

- You might miss signals
    
- It’s not atomic
    
- It’s not thread-safe
    

Blocking signals is the **correct POSIX mechanism**.

---

## 5️⃣ Subtle but important details

### ✔ Signals are not lost

They’re queued as _pending_ while blocked.

### ✔ Mask changes are atomic

No race window.

### ✔ Thread-specific

In multithreaded programs:

- `sigprocmask()` affects only the calling thread
    
- (POSIX recommends `pthread_sigmask()` instead)
    

---

## 6️⃣ One-sentence summary

> This code **temporarily blocks SIGINT**, runs a critical section safely, and then **restores the previous signal mask**, ensuring no signal is lost and no race occurs.

If you want, I can show:

- what happens if you forget to restore the mask
    
- how this differs from `SA_RESTART`
    
- how this behaves in multithreaded programs
Sure — here’s a **small, realistic example** that shows _exactly_ why that pattern is used and what happens when `SIGINT` (Ctrl-C) arrives.

---

## Goal of the example

We want to:

- increment a shared counter
    
- handle `SIGINT`
    
- **avoid the signal handler seeing the counter in a half-updated state**
    

---

## Example program

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

volatile sig_atomic_t got_sigint = 0;
int counter = 0;

void handler(int sig)
{
    got_sigint = 1;
}

int main(void)
{
    struct sigaction sa = {0};
    sa.sa_handler = handler;
    sigaction(SIGINT, &sa, NULL);

    for (;;) {
        sigset_t set, old;

        /* Block SIGINT */
        sigemptyset(&set);
        sigaddset(&set, SIGINT);
        sigprocmask(SIG_BLOCK, &set, &old);

        /* ---- critical section ---- */
        counter++;
        printf("counter = %d\n", counter);
        /* -------------------------- */

        /* Restore previous mask */
        sigprocmask(SIG_SETMASK, &old, NULL);

        if (got_sigint) {
            printf("SIGINT handled safely, counter = %d\n", counter);
            break;
        }

        sleep(1);
    }

    return 0;
}
```

---

## What happens at runtime

### Case 1: No signal

```
counter = 1
counter = 2
counter = 3
...
```

---

### Case 2: You press Ctrl-C during the critical section

1. `SIGINT` arrives
    
2. It is **blocked**, so:
    
    - it becomes _pending_
        
    - the handler does **not** run yet
        
3. The critical section finishes
    
4. `sigprocmask(SIG_SETMASK, &old)` unblocks SIGINT
    
5. The handler runs immediately
    
6. `got_sigint` is checked safely
    

Output:

```
counter = 4
SIGINT handled safely, counter = 4
```

✔ The handler never sees a partially updated `counter`

---

## What would go wrong _without_ blocking?

If you removed the `sigprocmask()` calls:

- SIGINT could interrupt **between** operations
    
- `printf()` could be interrupted
    
- shared state could be inconsistent
    
- undefined behavior becomes possible
    

---

## Key insight

Even though:

```c
counter++;
```

_looks_ atomic, it is not guaranteed to be:

- it may compile to multiple instructions
    
- a signal can interrupt in the middle
    

Blocking the signal removes that race.

---

## One-sentence takeaway

> This pattern ensures that a signal handler either runs **before or after** a critical section — never **during** it.

If you want, I can also show:

- the same example using the **self-pipe trick**
    
- a multithreaded version with `pthread_sigmask`
    
- how `SA_RESTART` changes behavior