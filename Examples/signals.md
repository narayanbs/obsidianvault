```c
#include <setjmp.h>
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

static sigjmp_buf env;

void myhandler(int sig) {
  write(STDOUT_FILENO, "sigfpe caught restarting\n", 26);
  siglongjmp(env, 1);
}

int main(void) {
  struct sigaction sa;
  sa.sa_handler = myhandler;
  sa.sa_flags = SA_RESTART;
  sigemptyset(&sa.sa_mask);

  if (sigaction(SIGFPE, &sa, NULL) < 0) {
    perror("sigaction");
  }

  if (sigsetjmp(env, 0) == 0) {
    int x = 20, y = 0;
    int z = x / y;
    printf("z is %d\n", z);
  } else {
    printf("Recovered from sigfpe\n");
  }
}

```

This code demonstrates **recovering from `SIGFPE` using `sigsetjmp()` / `siglongjmp()`**. A few important points:

 - `SIGFPE` is raised by the integer division `x / y` where `y == 0`.
- `sigsetjmp(env, 0)` saves the execution context. On the first pass it returns `0`.
- The signal handler calls `siglongjmp(env, 1)`, which jumps back to the `sigsetjmp()` call. This time it returns `1`, so `"Recovered from sigfpe"` is printed.
- `write()` is correctly used inside the signal handler. It is async-signal-safe, whereas `printf()` generally should **not** be called from a signal handler.
- `SA_RESTART` isn't particularly useful here because you're not trying to restart a blocking system call interrupted by `SIGFPE`.
- There is a small bug in the `write()` call: `"sigfpe caught restarting\n"` is **25 bytes**, not 26. Passing `26` may cause one byte beyond the intended string to be written. Use `sizeof(...) - 1` or the correct length.
- `sigsetjmp(env, 0)` means the signal mask isn't saved/restored. If you specifically want the signal mask restored by `siglongjmp()`, use `sigsetjmp(env, 1)`.
- Most importantly, this is **not a general-purpose way to recover from arbitrary arithmetic errors**. After a hardware-generated `SIGFPE`, jumping out of the faulting operation can be platform/compiler-dependent territory. For a simple demonstration it is useful, but production code should normally avoid relying on signals for ordinary divide-by-zero handling.

 One corrected version of the handler would be:

```c
void myhandler(int sig) {
    static const char msg[] = "SIGFPE caught, restarting\n";
    write(STDOUT_FILENO, msg, sizeof(msg) - 1);
    siglongjmp(env, 1);
}
```

 Also, `sig` is unused, so you could explicitly write `(void)sig;` to make that clear.

 Overall, **as a demonstration of `SIGFPE` \+ `sigsetjmp`/`siglongjmp`, the code is reasonable, but it needs a little cleanup and shouldn't be presented as a portable/general error-recovery pattern.**