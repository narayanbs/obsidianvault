In Linux (and Unix-like systems), the **`exec` family of functions** is used to **replace the current process image with a new program**. These functions are essential for process creation and program execution, especially when combined with `fork()`.

---

## 🔹 What does `exec` do?

- It **loads a new program into the current process**
    
- The **process ID (PID stays the same)**
    
- Code, data, heap, and stack are replaced
    
- Open file descriptors remain open unless marked `close-on-exec`
    
- If `exec` succeeds, **it never returns**
    
- If it returns, an **error occurred**
    

---

## 🔹 Why use `exec`?

Typically used like this:

1. `fork()` → create a child process
    
2. `exec()` → child runs a new program
    

This is how shells run commands.

---

## 🔹 The exec family functions

### 1. `execl`

```c
int execl(const char *path, const char *arg0, ..., NULL);
```

- **l** = list of arguments
    
- Arguments passed individually
    

Example:

```c
execl("/bin/ls", "ls", "-l", NULL);
```

---

### 2. `execlp`

```c
int execlp(const char *file, const char *arg0, ..., NULL);
```

- **p** = searches `PATH`
    
- No need for full path
    

Example:

```c
execlp("ls", "ls", "-l", NULL);
```

---

### 3. `execv`

```c
int execv(const char *path, char *const argv[]);
```

- **v** = vector (array of arguments)
    

Example:

```c
char *args[] = {"ls", "-l", NULL};
execv("/bin/ls", args);
```

---

### 4. `execvp`

```c
int execvp(const char *file, char *const argv[]);
```

- Combines **v + p**
    
- Most commonly used
    

Example:

```c
char *args[] = {"ls", "-l", NULL};
execvp("ls", args);
```

---

### 5. `execle`

```c
int execle(const char *path, const char *arg0, ..., NULL, char *const envp[]);
```

- **e** = custom environment
    
- You explicitly pass environment variables
    

Example:

```c
char *env[] = {"PATH=/bin", "USER=test", NULL};
execle("/bin/ls", "ls", "-l", NULL, env);
```

---

### 6. `execve` (core system call)

```c
int execve(const char *path, char *const argv[], char *const envp[]);
```

- The **actual system call**
    
- All other exec functions wrap around this
    

Example:

```c
char *args[] = {"ls", "-l", NULL};
char *env[] = {NULL};
execve("/bin/ls", args, env);
```

---

## 🔹 Summary table

|Function|Args Style|PATH Search|Custom Env|
|---|---|---|---|
|execl|list|❌|❌|
|execlp|list|✅|❌|
|execv|array|❌|❌|
|execvp|array|✅|❌|
|execle|list|❌|✅|
|execve|array|❌|✅|

---

## 🔹 Common mistakes

- Forgetting `NULL` at the end of arguments
    
- Assuming `exec()` returns on success
    
- Not passing program name as `argv[0]`
    

---

## 🔹 Simple fork + exec example

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {  // child
        execlp("ls", "ls", "-l", NULL);
        perror("exec failed");
    } else {
        printf("Parent process\n");
    }
    return 0;
}
```

---

If you want, I can:

- Explain **fork vs exec** in more detail
    
- Show **how a shell uses exec**
    
- Give **interview-oriented explanations**
    
- Draw a **process memory diagram**
    

Just tell me 😊