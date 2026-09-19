
 ## Linux `clone()`: How Processes and Threads Are Started

 A useful way to understand Linux process/thread creation is to distinguish **three layers**:

```
Application
    │
    ├── fork()
    ├── pthread_create()
    └── clone()          ← glibc wrapper
             │
             ▼
       clone / clone3    ← Linux system call
             │
             ▼
       kernel task creation
             │
             ▼
        task_struct
```

 The important idea is:

 > **Linux uses common kernel task-creation machinery for both processes and threads. The difference is primarily which resources the new task shares with its parent.**

---

 ## 1\. The glibc `clone()` wrapper

 glibc provides a convenient `clone()` function with an interface roughly like:

```
int clone(int (*fn)(void *),
          void *stack,
          int flags,
          void *arg, ...);
```

 The important arguments are:

```
fn      → function the new task should execute
stack   → stack for the new task
flags   → resources to share with the parent
arg     → argument passed to fn
```

 For example:

```
int child_function(void *arg)
{
    printf("Hello from child\n");
    return 0;
}

char *stack = malloc(STACK_SIZE);

clone(child_function,
      stack + STACK_SIZE,
      flags,
      NULL);
```

 The glibc wrapper provides a convenient abstraction:

```
"Create a task and start it by executing fn(arg)
 using this stack."
```

 This interface is particularly useful for creating **threads**, because a new thread needs a separately allocated stack.

---

 # 2\. The raw `clone` system call is different

 The Linux system call is a lower-level interface.

 Conceptually, the traditional `clone` syscall looks like:

```
clone(flags,
      child_stack,
      parent_tid,
      child_tid,
      tls);
```

 The exact syscall argument ordering is architecture-dependent, so this is best understood conceptually rather than as portable C.

 Notice something important:

```
glibc clone():

    fn
    stack
    flags
    arg

                ↓

raw syscall:

    flags
    child_stack
    parent_tid
    child_tid
    tls
```

 The **function pointer `fn` is not a fundamental argument to the Linux `clone` syscall**.

 The glibc wrapper uses `fn` to arrange the child's initial execution so that it starts by calling:

```
fn(arg)
```

 The kernel syscall itself is concerned with creating the task and establishing its execution state.

---

 # 3\. What does `clone()` actually create?

 At the kernel level, both processes and threads are represented by a `task_struct`.

 Conceptually:

```
                   clone()
                      │
                      ▼
                 new task
                      │
                      ▼
                 task_struct
```

 What makes the new task behave like a **thread** or a **process** is largely determined by the `flags`.

 For example:

```
                    New task
                       │
          ┌────────────┴────────────┐
          │                         │
     share resources           don't share
          │                         │
          ▼                         ▼
       Thread                    Process
```

---

 # 4\. Creating a thread

 A thread typically shares the parent's address space.

 Important flags include:

```
CLONE_VM
CLONE_FILES
CLONE_FS
CLONE_SIGHAND
CLONE_THREAD
CLONE_SETTLS
```

 For example, conceptually:

```
clone(
    thread_function,
    new_stack,
    CLONE_VM |
    CLONE_FILES |
    CLONE_FS |
    CLONE_SIGHAND |
    CLONE_THREAD |
    CLONE_SETTLS,
    arg
);
```

 The most important flag for memory sharing is:

```
CLONE_VM
```

 It means that the parent and child use the same virtual address space.

 So:

```
             Same address space
                    │
        ┌───────────┴───────────┐
        │                       │
     Thread A                Thread B
        │                       │
     stack A                 stack B
     registers               registers
     TLS                     TLS
```

 The threads share:

 - code
- global variables
- heap
- memory mappings

 but each thread has its own:

 - CPU register state
- stack
- thread-local storage
- scheduling state

---

 # 5\. Creating a process

 A traditional process has its own address space.

 The important distinction is that you **do not specify `CLONE_VM`**.

 Conceptually:

```
clone(
    SIGCHLD,
    0,
    NULL,
    ...
);
```

 This gives:

```
          Separate address spaces
                  │
       ┌──────────┴──────────┐
       │                     │
   Process A             Process B
       │                     │
   address space         address space
       │                     │
   stack A               stack B
```

 Initially, Linux can use **copy-on-write**, so the physical memory doesn't necessarily get copied immediately.

 The two processes logically have separate address spaces, even though physical pages may initially be shared.

---

 # 6\. Why does a thread need a stack argument?

 Suppose:

```
char *stack = malloc(1024 * 1024);
```

 and:

```
clone(thread_function,
      stack + 1024 * 1024,
      ...);
```

 The new thread needs its own execution stack:

```
Shared address space
┌───────────────────────────────┐
│ code                          │
│ globals                       │
│ heap                          │
│                               │
│ ┌───────────────┐             │
│ │ Thread A stack│             │
│ └───────────────┘             │
│                               │
│ ┌───────────────┐             │
│ │ Thread B stack│ ← new_stack │
│ └───────────────┘             │
└───────────────────────────────┘
```

 The stack is just memory.

 Linux doesn't need to know that `malloc()`ed memory is "a stack" in some special sense.

 The important thing is that the new task's initial stack pointer is set to an address inside that memory.

 Conceptually:

```
Child CPU state:

    RIP → thread_function
    RSP → new_stack
```

 The CPU then naturally uses that memory as its stack.

---

 # 7\. Why doesn't `fork()` need a stack argument?

 This is an important difference.

 With:

```
fork();
```

 the child does **not** start by calling a new function supplied by the caller.

 Instead, the child essentially continues from the point where the parent called `fork()`.

 For example:

```
pid_t pid = fork();

if (pid == 0) {
    printf("child");
}
```

 Conceptually:

```
Parent:

        fork()
          │
          ▼
    instruction after fork()
          │
          ▼
        continue

Child:

        fork()
          │
          ▼
    instruction after fork()
          │
          ▼
        continue
```

 The child's CPU state is derived from the parent's state.

 Therefore, it already has:

```
instruction pointer → after fork()
stack pointer        → parent's current stack position
```

 The child's address space is a copy-on-write version of the parent's address space.

 Therefore `fork()` doesn't need to ask the caller:

```
"Give me a new stack."
```

---

 # 8\. How does glibc implement `fork()`?

 This is where the raw syscall becomes relevant.

 glibc can implement `fork()` using the Linux clone-family syscall machinery.

 A simplified example of what you may encounter in glibc is:

```
INLINE_SYSCALL(
    clone,
    4,
    CLONE_CHILD_SETTID |
    CLONE_CHILD_CLEARTID |
    SIGCHLD,
    0,
    NULL,
    &THREAD_SELF->tid
);
```

 The `4` here means:

 > **Pass four arguments to the syscall.**

 It does **not** mean the stack size or stack address.

 The arguments are conceptually:

```
Argument 1:
    flags

Argument 2:
    child_stack

Argument 3:
    parent_tid

Argument 4:
    child_tid
```

 So:

```
flags       = CLONE_CHILD_SETTID |
              CLONE_CHILD_CLEARTID |
              SIGCHLD

child_stack = 0

parent_tid  = NULL

child_tid   = &THREAD_SELF->tid
```

 The important part for understanding `fork()` is:

```
child_stack = 0
```

 because fork-like semantics don't require the caller to provide a completely new stack.

---

 # 9\. `SIGCHLD` vs `CLONE_THREAD`

 Another useful distinction is the termination behavior.

 For a traditional process created with `fork()`, the parent expects normal child-process semantics, including `SIGCHLD` when the child terminates.

 Therefore you commonly see:

```
SIGCHLD
```

 in the clone flags.

 A thread, on the other hand, normally uses:

```
CLONE_THREAD
```

 to put the new task into the same thread group.

 So, very roughly:

```
Process-like:

    clone(...)
       │
       ├── no CLONE_VM
       ├── SIGCHLD
       └── separate address space
```

 versus:

```
Thread-like:

    clone(...)
       │
       ├── CLONE_VM
       ├── CLONE_THREAD
       ├── CLONE_SIGHAND
       ├── CLONE_FILES
       └── separate stack
```

 The actual flag combinations used by glibc/pthreads are more extensive, but this captures the core idea.

---

 # 10\. What about `clone3()`?

 Modern Linux also provides `clone3()`, which is a newer version of the interface.

 Instead of passing many positional arguments, you provide a structure:

```
struct clone_args {
    __u64 flags;
    __u64 pidfd;
    __u64 child_tid;
    __u64 parent_tid;
    __u64 exit_signal;
    __u64 stack;
    __u64 stack_size;
    __u64 tls;
    ...
};
```

 This makes the interface easier to extend.

 Conceptually:

```
clone3()
   │
   ▼
clone_args
   │
   ├── flags
   ├── stack
   ├── stack_size
   ├── parent_tid
   ├── child_tid
   ├── tls
   └── exit_signal
```

---

 # 11\. The complete picture

 The easiest mental model is:

```
                         Linux task creation
                                  │
                         clone / clone3 syscall
                                  │
                                  ▼
                           new task_struct
                                  │
                 ┌────────────────┴────────────────┐
                 │                                 │
           shared resources                  independent resources
                 │                                 │
                 ▼                                 ▼
              Thread                            Process
```

 More specifically:

```
THREAD
────────────────────────────────────────

pthread_create()
       │
       ▼
thread library / glibc
       │
       ▼
clone() / clone3()
       │
       ├── CLONE_VM
       ├── CLONE_THREAD
       ├── CLONE_SIGHAND
       ├── CLONE_FILES
       ├── CLONE_SETTLS
       └── new stack
       │
       ▼
new task sharing parent's resources
```

 versus:

```
PROCESS
────────────────────────────────────────

fork()
       │
       ▼
glibc
       │
       ▼
clone-family syscall
       │
       ├── no CLONE_VM
       ├── SIGCHLD
       └── no explicitly supplied new stack
       │
       ▼
new task with separate address space
       │
       ▼
copy-on-write memory
```

 ## The key takeaway

 Don't think of Linux as having two completely different kernel mechanisms:

```
process creation mechanism
thread creation mechanism
```

 Instead, think:

```
                  CREATE A TASK
                       │
                       ▼
                    clone()
                       │
          ┌────────────┴────────────┐
          │                         │
       share VM                 don't share VM
          │                         │
          ▼                         ▼
       thread                    process
```

 The **`clone()` syscall is a general task-creation primitive**. Its flags determine which resources are shared, while the userspace API around it (`fork()`, `pthread_create()`, glibc `clone()`, etc.) provides higher-level semantics appropriate to the use case.