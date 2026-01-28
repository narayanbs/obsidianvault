 Here are the **common environment variables** you’ll see when compiling, linking, and running C/C++ programs, especially on Unix/Linux systems.

I’ll group them by **purpose**, because that’s how they actually matter in practice.

---

## 🔧 Compiler selection

These decide _which compiler_ gets used.

- **`CC`**  
    C compiler (e.g. `gcc`, `clang`)
    
    ```sh
    export CC=gcc
    ```
    
- **`CXX`**  
    C++ compiler (e.g. `g++`, `clang++`)
    
    ```sh
    export CXX=g++
    ```
    

Commonly used by `make`, `cmake`, `autotools`, etc.

---

## 🧠 Compiler flags

These inject flags into the compilation step.

- **`CFLAGS`** – C compiler flags
    
    ```sh
    export CFLAGS="-O2 -Wall -Wextra"
    ```
    
- **`CXXFLAGS`** – C++ compiler flags
    
    ```sh
    export CXXFLAGS="-O2 -std=c++20"
    ```
    
- **`CPPFLAGS`** – Preprocessor flags (shared by C & C++)  
    Mostly include paths and macros:
    
    ```sh
    export CPPFLAGS="-I/usr/local/include -DMY_FEATURE"
    ```
    

> Rule of thumb:
> 
> - `CPPFLAGS`: `-I`, `-D`
>     
> - `CFLAGS/CXXFLAGS`: warnings, optimizations, language standards
>     

---

## 🔗 Linker flags

These affect the **linking** stage.

- **`LDFLAGS`**  
    Library paths and linker options:
    
    ```sh
    export LDFLAGS="-L/usr/local/lib -Wl,-rpath,/usr/local/lib"
    ```
    
- **`LIBS`**  
    Explicit libraries:
    
    ```sh
    export LIBS="-lm -lpthread"
    ```
    

---

## 📚 Library search paths (runtime & link-time)

### Runtime dynamic linker

- **`LD_LIBRARY_PATH`** (Linux)
    
- **`DYLD_LIBRARY_PATH`** (macOS, restricted by SIP)
    
- **`LIBRARY_PATH`** (used by GCC at link time, _not_ runtime)
    

Example:

```sh
export LD_LIBRARY_PATH=/opt/mylib/lib:$LD_LIBRARY_PATH
```

⚠️ `LD_LIBRARY_PATH` affects **runtime**, not just builds.

---

## 📁 Include paths

While usually passed via flags, these sometimes appear as env vars:

- **`CPATH`**  
    Global include path for GCC/Clang:
    
    ```sh
    export CPATH=/opt/mylib/include
    ```
    
- **`C_INCLUDE_PATH`**
    
- **`CPLUS_INCLUDE_PATH`**
    

Used automatically by GCC-like compilers.

---

## 🧵 Toolchain & binutils

Where tools are found.

- **`PATH`**  
    Controls which compiler, linker, and tools run:
    
    ```sh
    export PATH=/opt/toolchain/bin:$PATH
    ```
    
- **`AR`** – static library archiver
    
- **`AS`** – assembler
    
- **`LD`** – linker
    
- **`NM`, `STRIP`, `OBJDUMP`**, etc.
    

Mostly relevant in cross-compilation.

---

## 🌍 Locale & behavior (less obvious but important)

- **`LANG` / `LC_ALL`**  
    Affects compiler diagnostics and build scripts:
    
    ```sh
    export LC_ALL=C
    ```
    

This is surprisingly important for reproducible builds.

---

## 🏗️ Build system–specific (very common)

### CMake

- **`CMAKE_PREFIX_PATH`**
    
- **`CMAKE_LIBRARY_PATH`**
    
- **`CMAKE_INCLUDE_PATH`**
    
- **`PKG_CONFIG_PATH`**
    

### pkg-config

- **`PKG_CONFIG_PATH`**
    
- **`PKG_CONFIG_LIBDIR`**
    

Example:

```sh
export PKG_CONFIG_PATH=/opt/mylib/lib/pkgconfig
```

---

## 🔁 Cross-compilation

You’ll often see:

- **`SYSROOT`**
    
- **`CROSS_COMPILE`**
    
- **`TARGET`**, **`HOST`**, **`BUILD`**
    

Example:

```sh
export CROSS_COMPILE=aarch64-linux-gnu-
```

---

## 🧩 Quick “most common” short list

If you only remember a few, remember these:

- `CC`, `CXX`
    
- `CFLAGS`, `CXXFLAGS`, `CPPFLAGS`
    
- `LDFLAGS`
    
- `LD_LIBRARY_PATH`
    
- `PATH`
    
- `PKG_CONFIG_PATH`
    

---

If you want, I can:

- explain **which ones are read by GCC vs Clang**
    
- show a **minimal clean env** for reproducible builds
    
- compare **`LD_LIBRARY_PATH` vs rpath vs RUNPATH**
    
- or tailor this to **Linux vs macOS vs cross-compile**
    

Just say the word 👀