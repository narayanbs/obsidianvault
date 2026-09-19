# Creating Static Libraries and Dynamic Shared Objects on Linux Using GCC

On Linux, GCC can be used to compile source files into object files and then package those object files into either **static libraries** or **dynamic shared objects (DSOs)**. Understanding both approaches is important because the way a library is created affects how it is linked and loaded by an application.

## 1. Compilation Process

Consider a simple library containing two source files:

```text
math.c
math.h
```

The header file contains the function declarations:

```c
#ifndef MATH_H
#define MATH_H

int add(int a, int b);
int multiply(int a, int b);

#endif
```

The implementation can be placed in `math.c`:

```c
#include "math.h"

int add(int a, int b)
{
    return a + b;
}

int multiply(int a, int b)
{
    return a * b;
}
```

The first step is to compile the source file into an object file:

```bash
gcc -c math.c -o math.o
```

This produces:

```text
math.o
```

The `-c` option tells GCC to compile the source code but **not perform the final linking step**.

---

# 2. Creating a Static Library

A static library is an archive containing one or more object files. On Linux, static libraries normally have the `.a` extension and are commonly named using the convention:

```text
lib<name>.a
```

For example:

```text
libmath.a
```

The `ar` utility is used to create the archive:

```bash
ar rcs libmath.a math.o
```

Here:

- `r` adds or replaces files in the archive.
- `c` creates the archive if it doesn't already exist.
- `s` creates an index of the archive's symbols.

You now have:

```text
math.c
math.h
math.o
libmath.a
```

## 3. Linking a Static Library

Suppose the application is:

```c
#include <stdio.h>
#include "math.h"

int main(void)
{
    printf("%d\n", add(10, 20));
    printf("%d\n", multiply(5, 6));

    return 0;
}
```

Save it as `main.c`.

Compile and link it against the static library:

```bash
gcc main.c -L. -lmath -o app
```

The options mean:

```text
-L.       Search the current directory for libraries
-lmath    Look for libmath.a or libmath.so
-o app    Create an executable named app
```

If `libmath.a` is selected, the relevant object code from the archive is copied into the executable during linking.

The resulting program therefore does not need `libmath.a` at runtime.

---

# 4. Creating a Dynamic Shared Object

A dynamic shared object is the Linux equivalent of what is commonly called a shared library. It normally has the `.so` extension.

For example:

```text
libmath.so
```

To create one, the source code must first be compiled as **position-independent code (PIC)**:

```bash
gcc -fPIC -c math.c -o math.o
```

The important option here is:

```text
-fPIC
```

which generates position-independent code suitable for use in a shared object.

The shared object can then be created with:

```bash
gcc -shared -o libmath.so math.o
```

The `-shared` option tells GCC to produce a shared library rather than a normal executable.

You now have:

```text
math.c
math.h
math.o
libmath.so
```

---

# 5. Linking an Application with a Shared Library

The application can be linked against the shared library using:

```bash
gcc main.c -L. -lmath -o app
```

If both of these exist:

```text
libmath.a
libmath.so
```

GCC's normal library search behavior will generally select the shared library when dynamically linking.

You can check which shared libraries an executable requires using:

```bash
ldd ./app
```

You may see something similar to:

```text
libmath.so => not found
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
```

The `not found` occurs because the dynamic loader does not normally search the current directory for shared libraries.

For testing, you can tell the loader to search the current directory:

```bash
LD_LIBRARY_PATH=. ./app
```

---

# 6. Using rpath

Instead of setting `LD_LIBRARY_PATH`, you can embed a runtime library search path into the executable.

For example:

```bash
gcc main.c -L. -Wl,-rpath,'$ORIGIN' -lmath -o app
```

`$ORIGIN` refers to the directory containing the executable.

This allows the following layout:

```text
project/
├── app
└── libmath.so
```

and you can simply run:

```bash
./app
```

The dynamic loader will look for `libmath.so` in the directory containing `app`.

---

# 7. Static vs Dynamic Libraries

The fundamental difference is **when the library's code becomes part of the application**.

With a static library:

```text
             Linking
main.o ───────────────┐
                      ├──> app
libmath.a ────────────┘
```

The required library code is incorporated into the executable during the link step.

With a shared library:

```text
             Linking
main.o ───────────────┐
                      ├──> app ───> libmath.so
libmath.so ───────────┘
```

The executable contains information about the shared library it needs, and the dynamic loader loads that library when the program starts.

---

# 8. Complete Example

A simple project could look like this:

```text
project/
├── main.c
├── math.c
├── math.h
├── libmath.a
└── libmath.so
```

Create the object file for the static library:

```bash
gcc -c math.c -o math.o
```

Create the static library:

```bash
ar rcs libmath.a math.o
```

Create position-independent code for the shared library:

```bash
gcc -fPIC -c math.c -o math_pic.o
```

Create the shared library:

```bash
gcc -shared -o libmath.so math_pic.o
```

Build the application dynamically:

```bash
gcc main.c -L. -lmath -Wl,-rpath,'$ORIGIN' -o app
```

Run it:

```bash
./app
```

Check its dependencies:

```bash
ldd ./app
```

---

# 9. Explicitly Selecting Static or Dynamic Libraries

When both `libmath.a` and `libmath.so` exist, you can explicitly control which type GCC uses.

To statically link `libmath` while keeping the normal system libraries dynamic:

```bash
gcc main.c -L. -Wl,-Bstatic -lmath -Wl,-Bdynamic -o app
```

This is different from:

```bash
gcc -static main.c -L. -lmath -o app
```

The latter requests a **fully static link**, meaning GCC attempts to statically link the system libraries as well.

You can also directly specify the archive:

```bash
gcc main.c ./libmath.a -o app
```

or the shared object:

```bash
gcc main.c ./libmath.so -Wl,-rpath,'$ORIGIN' -o app
```

---

# 10. Summary

The typical commands are:

```bash
# Compile
gcc -c math.c -o math.o

# Create static library
ar rcs libmath.a math.o

# Compile as position-independent code
gcc -fPIC -c math.c -o math_pic.o

# Create shared library
gcc -shared -o libmath.so math_pic.o

# Link dynamically
gcc main.c -L. -lmath -o app

# Link static libmath, dynamic system libraries
gcc main.c -L. -Wl,-Bstatic -lmath -Wl,-Bdynamic -o app

# Fully static executable
gcc -static main.c -L. -lmath -o app
```

The most important concepts to remember are:

- **`.a`** → static archive, created with `ar`.
- **`.so`** → dynamic shared object, created with `gcc -shared`.
- **`-fPIC`** → generates position-independent code for shared libraries.
- **`-L`** → adds a library search directory.
- **`-lfoo`** → searches for `libfoo.so` or `libfoo.a`.
- **`-static`** → requests static linking.
- **`-Wl,-Bstatic` / `-Wl,-Bdynamic`** → lets you control static/dynamic selection for particular libraries.
- **`ldd`** → shows the dynamic dependencies of an executable.

--------------------
# Static linked binary and Dynamic linked binary

Yes. The key distinction is how the executable is linked against its libraries.

Suppose you have:
~~~cpp 
main.c
libfoo.a       # static library
libfoo.so      # shared library
~~~


and main.c calls functions from foo.

### Static-linked binary

To link everything statically:
```cpp
gcc -static main.c -L. -lfoo -o myprogram
```


This tells GCC:

*  -static → prefer/use static libraries for linking
*  -L. → look for libraries in the current directory
*  -lfoo → link against libfoo.a
*  -o myprogram → output executable

You can verify it with:
```cpp
ldd ./myprogram
```


A fully static binary will typically say:
~~~
not a dynamic executable
~~~


You can also inspect it with:
~~~cpp
file ./myprogram
~~~

### Dynamically linked binary

For dynamic linking:
~~~cpp
gcc main.c -L. -lfoo -o myprogram
~~~


GCC will normally prefer libfoo.so over libfoo.a.

You can verify:
~~~ cpp
ldd ./myprogram
~~~


You should see something like:
~~~cpp
libfoo.so => ./libfoo.so
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6
...
~~~


However, because libfoo.so is in the current directory, the loader may not automatically find it when you run the program. For testing:
~~~cpp
LD_LIBRARY_PATH=. ./myprogram
~~~


Or embed a runtime search path when linking:
~~~cpp
gcc main.c -L. -Wl,-rpath,'$ORIGIN' -lfoo -o myprogram
~~~


Then:
~~~cpp
./myprogram
~~~


will look for libfoo.so in the same directory as the executable.

One important subtlety

There are actually two different meanings of "static binary":

1. Statically link your own library

If you want your libfoo.a statically linked but still want glibc dynamically linked:
~~~cpp
gcc main.c -L. -Wl,-Bstatic -lfoo -Wl,-Bdynamic -o myprogram
~~~


Result:
~~~
myprogram
├── libfoo.a   ← embedded into executable
└── libc.so    ← dynamically loaded
~~~


This is often what people mean when they say "statically link my library."

2. Completely static executable
~~~cpp
gcc -static main.c -L. -lfoo -o myprogram
~~~


Result:
~~~
myprogram
├── libfoo.a   ← embedded
├── libc.a     ← embedded
├── libpthread.a / etc. ← embedded as applicable
└── no .so dependencies
~~~


So the basic rule is:

# Fully static
~~~cpp
gcc -static main.c -L. -lfoo -o app
~~~

# Fully dynamic
~~~cpp
gcc main.c -L. -lfoo -o app
~~~

# Static libfoo, dynamic system libraries
~~~cpp
gcc main.c -L. -Wl,-Bstatic -lfoo -Wl,-Bdynamic -o app
~~~


