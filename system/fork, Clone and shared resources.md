Below is a **clear, OS-level explanation** that ties together **`fork()`**, **copy-on-write (COW)**, **what is duplicated vs shared**, and how **`clone()`** gives fine-grained control in Linux.

---

## 1. What happens to memory when a process is forked?

When a process calls **`fork()`**, the kernel creates a **child process** that initially looks _almost exactly the same_ as the parent.

Conceptually, it seems like:

> “All memory is duplicated”

But in reality, **Linux does NOT immediately copy memory**. That would be slow and wasteful.

Instead, Linux uses **Copy-On-Write (COW)**.

---

## 2. Copy-On-Write (COW): the key idea

### Initial state after `fork()`

- Parent and child:
    
    - Have **separate virtual address spaces**
        
    - But **point to the same physical memory pages**
        
- Those pages are marked **read-only**
    

So initially:

```
Parent virtual memory ─┐
                        ├── Physical page (shared, read-only)
Child virtual memory ──┘
```

### When does copying actually happen?

Only **when one process writes** to a shared page.

1. Process attempts to write
    
2. CPU triggers a **page fault**
    
3. Kernel:
    
    - Allocates a new physical page
        
    - Copies the original page’s contents
        
    - Updates the page table **only for the writing process**
        
4. The other process continues using the old page
    

This is **Copy-On-Write**.

### Why COW is powerful

- Fast `fork()` (no immediate copying)
    
- Efficient memory usage
    
- Works great for patterns like:
    
    ```c
    fork();
    exec();
    ```
    
    (common in shells)
    

---

## 3. What exactly gets duplicated vs shared in `fork()`?

### Virtual memory

|Component|After fork|
|---|---|
|Virtual address space|Duplicated (separate page tables)|
|Physical memory pages|Shared initially (COW)|
|Stack|Shared via COW|
|Heap|Shared via COW|
|Global variables|Shared via COW|
|Code (text segment)|Shared (read-only anyway)|

> Important: **“Duplicated” usually means virtual mappings, not physical RAM**

---

### CPU & process state

|Item|Parent vs Child|
|---|---|
|Process ID (PID)|Different|
|Parent PID|Child points to parent|
|Registers|Copied|
|Program counter|Same instruction (fork returns)|
|Return value of `fork()`|0 in child, child PID in parent|

---

### File descriptors

- **Shared**
    
- Parent and child:
    
    - Point to the **same open file table entries**
        
    - Share file offset and flags
        

Example:

```c
fork();
write(fd, "A", 1);
```

→ Both processes advance the same file offset.

---

### Other kernel resources

|Resource|Behavior|
|---|---|
|Environment variables|Copied|
|Signal handlers|Copied|
|Memory mappings (`mmap`)|Copied (COW applies)|
|Locks|NOT duplicated safely|
|Threads|Only the calling thread is copied|

---

## 4. Why `fork()` is limited

`fork()` gives you:

- **Almost everything duplicated**
    
- **Almost everything shared only temporarily**
    

But sometimes you want:

- Shared memory permanently
    
- Shared file system context
    
- Shared signal handlers
    
- Thread-like behavior
    

This is where **`clone()`** comes in.

---

## 5. `clone()` system call: controlled sharing

`clone()` is the **low-level primitive** behind:

- `fork()`
    
- `vfork()`
    
- `pthread_create()`
    

### Key idea

> `clone()` lets you **choose what is shared** between parent and child.

Prototype (simplified):

```c
clone(function, stack, flags, arg);
```

---

## 6. `clone()` flags: what can be shared?

### Memory

|Flag|Meaning|
|---|---|
|`CLONE_VM`|Share the **same address space**|
|(no flag)|Separate address spaces (like fork)|

- With `CLONE_VM`: **no COW**
    
- Writes are immediately visible to both processes
    

This is how **threads** work.

---

### Filesystem & file descriptors

|Flag|Shared|
|---|---|
|`CLONE_FILES`|File descriptor table|
|`CLONE_FS`|Current working directory, root|
|`CLONE_IO`|I/O context|

---

### Signals & execution

|Flag|Effect|
|---|---|
|`CLONE_SIGHAND`|Signal handlers|
|`CLONE_THREAD`|Same thread group (same PID)|
|`CLONE_PARENT`|Same parent|

---

### Identity & namespaces

|Flag|Purpose|
|---|---|
|`CLONE_NEWNS`|Mount namespace|
|`CLONE_NEWPID`|PID namespace|
|`CLONE_NEWNET`|Network namespace|
|`CLONE_NEWUSER`|User namespace|

These are the foundation of **containers (Docker)**.

---

## 7. fork vs clone vs threads (big picture)

|Feature|fork()|clone()|pthread|
|---|---|---|---|
|Address space|COW|Configurable|Shared|
|File descriptors|Shared|Optional|Shared|
|Signal handlers|Copied|Optional|Shared|
|PID|New|Configurable|Same|
|Typical use|New process|Custom behavior|Threads|

---

## 8. Why Linux designed it this way

- **`fork()`**: simple, safe, POSIX-compatible
    
- **COW**: performance without semantic change
    
- **`clone()`**: maximum flexibility
    
- **Threads = processes with shared resources**
    

In Linux:

> **Threads are just processes that share more things**

---

## 9. One-sentence summary

- **`fork()`** duplicates the _process view_, not memory immediately
    
- **Copy-On-Write** delays physical copying until a write occurs
    
- **`clone()`** lets you explicitly choose which resources are shared
    
- **Threads** are created by sharing memory, files, and signals via `clone()`
    

---

If you want, I can also:

- Walk through **page tables step-by-step**
    
- Explain **`vfork()` vs `fork()`**
    
- Show **how pthreads use `clone()` internally**
    
- Draw a **memory layout diagram**
    

Just tell me 👍