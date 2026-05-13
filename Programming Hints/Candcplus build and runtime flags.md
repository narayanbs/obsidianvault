 
Here are the main environment variables you’ll encounter in C/C++ compilation and execution on Unix-like systems, especially with GNU Compiler Collection and Clang.

---

# Include Path Variables

These affect where headers (`#include`) are searched.

| Variable                | Purpose                              | Applies To       |
| ----------------------- | ------------------------------------ | ---------------- |
| `CPATH`                 | Extra include dirs for all languages | C, C++, ObjC     |
| `C_INCLUDE_PATH`        | Extra include dirs for C             | `gcc`            |
| `CPLUS_INCLUDE_PATH`    | Extra include dirs for C++           | `g++`, `clang++` |


Example:

```bash
export CPLUS_INCLUDE_PATH=/opt/mylib/include
```

Equivalent to:

```bash
g++ -I/opt/mylib/include
```

---

# Library Search Variables (Link Time)

These affect where libraries are found during linking.

|Variable|Purpose|
|---|---|
|`LIBRARY_PATH`|Extra directories searched by linker during compilation|

Example:

```bash
export LIBRARY_PATH=/opt/mylib/lib
gcc main.c -lfoo
```

Equivalent to:

```bash
gcc main.c -L/opt/mylib/lib -lfoo
```

---

# Runtime Dynamic Loader Variables

These affect where shared libraries are searched when the executable runs.

| Variable            | Platform       | Purpose                            |     |
| ------------------- | -------------- | ---------------------------------- | --- |
| `LD_LIBRARY_PATH`   | Linux/ELF      | Runtime shared library search path |     |
| `DYLD_LIBRARY_PATH` | macOS          | Runtime shared library search path |     |
| `PATH`              | Windows (DLLs) | DLL lookup path                    |     |

Example:

```bash
export LD_LIBRARY_PATH=/opt/mylib/lib
./app
```

---

# Compiler Selection Variables

Commonly used by build systems and scripts.

|Variable|Meaning|
|---|---|
|`CC`|C compiler|
|`CXX`|C++ compiler|
|`CPP`|C preprocessor|
|`LD`|Linker|
|`AR`|Static library archiver|
|`AS`|Assembler|
|`NM`|Symbol table tool|
|`RANLIB`|Archive indexer|
|`STRIP`|Symbol stripper|

Examples:

```bash
export CC=clang
export CXX=clang++
```

Used by:

- CMake
    
- GNU Make
    
- Autotools
    
- Meson
    

---

# Build Flag Variables

Very commonly used in Makefiles and build systems.

|Variable|Meaning|
|---|---|
|`CFLAGS`|Extra flags for C compiler|
|`CXXFLAGS`|Extra flags for C++ compiler|
|`CPPFLAGS`|Preprocessor flags (`-I`, `-D`)|
|`LDFLAGS`|Linker flags (`-L`, `-Wl,...`)|
|`LDLIBS`|Libraries (`-lm`, `-lpthread`)|

Example:

```bash
export CFLAGS="-O2 -Wall"
export LDFLAGS="-L/opt/mylib/lib"
```

---

# pkg-config Variables

Used heavily in modern Linux development.

|Variable|Meaning|
|---|---|
|`PKG_CONFIG_PATH`|Extra `.pc` metadata search dirs|
|`PKG_CONFIG_LIBDIR`|Override default pkg-config dirs|

Example:

```bash
export PKG_CONFIG_PATH=/opt/mylib/lib/pkgconfig
pkg-config --cflags gtk4
```

---

# Toolchain / Sysroot Variables

Important for cross-compilation.

|Variable|Meaning|
|---|---|
|`SYSROOT`|Alternate root filesystem|
|`CMAKE_SYSROOT`|CMake sysroot|
|`SDKROOT`|Apple SDK selection|

---

# Debugging / Loader Variables

Useful for diagnosing runtime linking issues.

|Variable|Meaning|
|---|---|
|`LD_DEBUG`|Dynamic loader debug output|
|`LD_PRELOAD`|Force-load shared libraries before others|

Example:

```bash
LD_DEBUG=libs ./app
```

or:

```bash
LD_PRELOAD=./mylib.so ./app
```

---

# The Most Important Ones in Practice

If you only remember a few:

|Variable|What it controls|
|---|---|
|`PATH`|Executable search path|
|`CC` / `CXX`|Which compiler to use|
|`CFLAGS` / `CXXFLAGS`|Compiler options|
|`CPPFLAGS`|Include paths / defines|
|`LDFLAGS`|Linker options|
|`LD_LIBRARY_PATH`|Runtime shared library lookup|
|`PKG_CONFIG_PATH`|Package metadata lookup|

---

# Mental Model

Think of compilation as 3 phases:

## 1. Preprocessing

Headers searched via:

- `-I`
    
- `CPATH`
    
- `C_INCLUDE_PATH`
    
- `CPLUS_INCLUDE_PATH`
    

---

## 2. Linking

Libraries searched via:

- `-L`
    
- `LIBRARY_PATH`
    

---

## 3. Runtime

Shared libraries searched via:

- embedded RPATH/RUNPATH
    
- `LD_LIBRARY_PATH`
    
- system library dirs (`/lib`, `/usr/lib`)


