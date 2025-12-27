Below is a **practical reference list** of the important *standard* predefined macros in **C** and **C++**—the ones you commonly use to detect the compilation language, standard version, or environment.

---

# ✅ **C++ Standard Macros**

### **Language / Standard Detection**

| Macro         | Meaning                                                                                                 |
| ------------- | ------------------------------------------------------------------------------------------------------- |
| `__cplusplus` | Required by the C++ standard. Indicates you are compiling as C++, and encodes the C++ language version. |

Values (typical):

* C++98/03 → `199711L`
* C++11 → `201103L`
* C++14 → `201402L`
* C++17 → `201703L`
* C++20 → `202002L`
* C++23 → `202302L`

---

### **Environment / Implementation (C++ but not strictly C++)**

These are widely supported but not part of the core ISO standard; they are de-facto standard:

| Macro                   | Meaning                               |
| ----------------------- | ------------------------------------- |
| `_WIN32`                | Windows (32 & 64 bit)                 |
| `_WIN64`                | Windows 64-bit                        |
| `__linux__`             | Linux                                 |
| `__unix__`              | Unix-like systems                     |
| `__APPLE__`, `__MACH__` | macOS / iOS                           |
| `__GNUC__`              | GCC / Clang compatibility mode        |
| `__clang__`             | Clang compiler                        |
| `_MSC_VER`              | Microsoft Visual C++ compiler version |

---

# ✅ **C Standard Macros**

### **Language / Standard Detection**

| Macro              | Meaning                                                            |
| ------------------ | ------------------------------------------------------------------ |
| `__STDC__`         | Defined as `1` in strictly conforming C mode (not required in C++) |
| `__STDC_VERSION__` | C standard version (C99+, optional in C90).                        |

Typical values:

* C90 → *undefined* (or `199409L` with amendments)
* C99 → `199901L`
* C11 → `201112L`
* C17 (C18) → `201710L`
* C23 → `202311L`

---

### **Optional C Feature Macros (from C11+)**

These indicate optional C standard features:

| Macro                 | Meaning                                       |
| --------------------- | --------------------------------------------- |
| `__STDC_NO_ATOMICS__` | Atomics not supported                         |
| `__STDC_NO_COMPLEX__` | Complex numbers not supported                 |
| `__STDC_NO_THREADS__` | C11 threads not supported                     |
| `__STDC_NO_VLA__`     | Variable-length arrays not supported (in C23) |

---

# ✅ **Macros Common to Both C and C++ (Defined by the Preprocessor Itself)**

| Macro             | Meaning                                          |
| ----------------- | ------------------------------------------------ |
| `__FILE__`        | Current file name (string literal)               |
| `__LINE__`        | Current line number                              |
| `__DATE__`        | Compilation date                                 |
| `__TIME__`        | Compilation time                                 |
| `__func__`        | Current function name (C99/C++11)                |
| `__STDC_HOSTED__` | `1` for hosted environment, `0` for freestanding |

---

# 📌 **Special Case: C++ Compatibility With C Headers**

C++ defines one extra macro not in C:

| Macro                    | Meaning                                                                                    |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| `__STDC_LIMIT_MACROS`    | Enables limit macros from `<stdint.h>` in C++ (when defined before including `<stdint.h>`) |
| `__STDC_CONSTANT_MACROS` | Enables constant macros (e.g., `INT64_C`) in C++                                           |

These are used with old C headers to access constants.

---

# ⭐ Summary

### **Core must-know macros:**

**C++ only**

* `__cplusplus`

**C only**

* `__STDC__`
* `__STDC_VERSION__`

**Common to both**

* `__FILE__`, `__LINE__`, `__DATE__`, `__TIME__`, `__func__`, `__STDC_HOSTED__`

**Environment / Compiler**

* `_WIN32`, `__linux__`, `__APPLE__`, `__GNUC__`, `__clang__`, `_MSC_VER`

---

If you'd like, I can also give you:
✅ A table of **compiler-specific** feature macros
✅ A list of **C++ library feature test macros** like `__cpp_concepts`, `__cpp_lib_format`, etc.
Just tell me!

Great — here is the **complete**, up-to-date list of **C++ feature-test macros** and **C standard feature macros**, plus important **compiler-specific** macros.
These are the macros used to check whether specific **language** or **library** features are available.

---

# 🟦 **1. C++ Language Feature-Test Macros (`__cpp_…`)**

These are standardized by ISO C++.
They are defined in **all conforming C++ compilers** for supported features.

### 🔹 *Core language feature macros*

| Macro                           | Meaning                                                  | First standard |
| ------------------------------- | -------------------------------------------------------- | -------------- |
| `__cpp_constexpr`               | `constexpr` support (value increases in newer standards) | C++11          |
| `__cpp_static_assert`           | `static_assert`                                          | C++11          |
| `__cpp_decltype`                | `decltype`                                               | C++11          |
| `__cpp_range_based_for`         | Range-based for loops                                    | C++11          |
| `__cpp_unicode_characters`      | Unicode char and string literals                         | C++11          |
| `__cpp_raw_strings`             | Raw string literals                                      | C++11          |
| `__cpp_lambdas`                 | Lambdas                                                  | C++11          |
| `__cpp_rvalue_reference`        | Rvalue references (`&&`)                                 | C++11          |
| `__cpp_variadic_templates`      | Variadic templates                                       | C++11          |
| `__cpp_initializer_lists`       | `{}` initializer lists                                   | C++11          |
| `__cpp_attributes`              | Standard attributes (`[[…]]`)                            | C++11          |
| `__cpp_nsdmi`                   | Default member initializers                              | C++11          |
| `__cpp_delegating_constructors` | Delegating constructors                                  | C++11          |
| `__cpp_threadsafe_static_init`  | Thread-safe static local initialization                  | C++11          |
| `__cpp_ref_qualifiers`          | Ref-qualified member functions                           | C++11          |
| `__cpp_user_defined_literals`   | UDLs (`"x"_suffix`)                                      | C++11          |
| `__cpp_alias_templates`         | Template aliases                                         | C++11          |
| `__cpp_exception_ptr`           | `std::exception_ptr`                                     | C++11          |

### 🔹 *C++14*

| Macro                      | Meaning                           |
| -------------------------- | --------------------------------- |
| `__cpp_binary_literals`    | `0b1010`                          |
| `__cpp_aggregate_nsdmi`    | NSDMI in aggregates               |
| `__cpp_init_captures`      | Lambda init-capture (`[x = f()]`) |
| `__cpp_generic_lambdas`    | Generic lambdas (`auto` params)   |
| `__cpp_variable_templates` | Variable templates                |

### 🔹 *C++17*

| Macro                          | Meaning                           |
| ------------------------------ | --------------------------------- |
| `__cpp_fold_expressions`       | Fold expressions                  |
| `__cpp_noexcept_function_type` | `noexcept` as part of type system |
| `__cpp_inline_variables`       | Inline variables                  |
| `__cpp_if_constexpr`           | `if constexpr`                    |
| `__cpp_deduction_guides`       | CTAD guides                       |
| `__cpp_capture_star_this`      | `[*this]` lambda capture          |
| `__cpp_structured_bindings`    | Structured bindings               |

### 🔹 *C++20*

| Macro                           | Meaning                        |
| ------------------------------- | ------------------------------ |
| `__cpp_concepts`                | Concepts (`requires`)          |
| `__cpp_constexpr_dynamic_alloc` | `constexpr` new/delete         |
| `__cpp_impl_coroutine`          | Coroutines                     |
| `__cpp_modules`                 | C++20 Modules                  |
| `__cpp_char8_t`                 | `char8_t` type                 |
| `__cpp_constexpr_virtual`       | Virtual functions in constexpr |
| `__cpp_lib_remove_cvref`        | `std::remove_cvref` available  |

### 🔹 *C++23*

| Macro                              | Meaning                             |
| ---------------------------------- | ----------------------------------- |
| `__cpp_multidimensional_subscript` | `obj[i,j]` syntax                   |
| `__cpp_auto_cast`                  | `auto(x)` cast                      |
| `__cpp_size_t_suffix`              | `123uz` suffix                      |
| `__cpp_if_consteval`               | `if consteval`                      |
| `__cpp_explicit_this_parameter`    | `this` as explicit object parameter |

---

# 🟩 **2. C++ Standard Library Feature Macros (`__cpp_lib_…`)**

C++ library feature availability is indicated by macros such as:

### C++11+

| Macro                           | Feature                    |
| ------------------------------- | -------------------------- |
| `__cpp_lib_chrono`              | `<chrono>`                 |
| `__cpp_lib_thread`              | `<thread>`                 |
| `__cpp_lib_atomic`              | `<atomic>`                 |
| `__cpp_lib_shared_ptr`          | `std::shared_ptr`          |
| `__cpp_lib_make_unique`         | `std::make_unique`         |
| `__cpp_lib_tuple`               | `std::tuple`               |
| `__cpp_lib_addressof_constexpr` | constexpr `std::addressof` |

### C++14+

| Macro                        | Feature                               |
| ---------------------------- | ------------------------------------- |
| `__cpp_lib_make_unique`      | Added earlier but stabilized in C++14 |
| `__cpp_lib_integer_sequence` | `std::integer_sequence`               |

### C++17+

| Macro                          | Feature             |
| ------------------------------ | ------------------- |
| `__cpp_lib_any`                | `std::any`          |
| `__cpp_lib_optional`           | `std::optional`     |
| `__cpp_lib_variant`            | `std::variant`      |
| `__cpp_lib_filesystem`         | `<filesystem>`      |
| `__cpp_lib_string_view`        | `std::string_view`  |
| `__cpp_lib_parallel_algorithm` | Parallel algorithms |

### C++20+

| Macro                        | Feature                     |
| ---------------------------- | --------------------------- |
| `__cpp_lib_format`           | `std::format`               |
| `__cpp_lib_ranges`           | `<ranges>`                  |
| `__cpp_lib_concepts`         | Concepts library            |
| `__cpp_lib_coroutine`        | Coroutine types             |
| `__cpp_lib_source_location`  | `std::source_location`      |
| `__cpp_lib_constexpr_vector` | constexpr vector operations |

### C++23+

| Macro                  | Feature           |
| ---------------------- | ----------------- |
| `__cpp_lib_expected`   | `std::expected`   |
| `__cpp_lib_print`      | `std::print`      |
| `__cpp_lib_spanstream` | `std::spanstream` |

---

# 🟧 **3. C Feature-Test Macros (`__STDC_…`)**

### C11 Optional Features

| Macro                 | Meaning                     |
| --------------------- | --------------------------- |
| `__STDC_NO_ATOMICS__` | `<stdatomic.h>` unsupported |
| `__STDC_NO_COMPLEX__` | `_Complex` unsupported      |
| `__STDC_NO_THREADS__` | `<threads.h>` unsupported   |

### C23

| Macro             | Meaning                    |
| ----------------- | -------------------------- |
| `__STDC_NO_VLA__` | VLAs removed from standard |

---

# 🟥 **4. Compiler-Specific Identification Macros**

These help detect the compiler:

### GCC & Clang

| Macro                                                        | Meaning                            |
| ------------------------------------------------------------ | ---------------------------------- |
| `__GNUC__`                                                   | GCC version (or Clang in GCC mode) |
| `__GNUC_MINOR__`                                             | Minor version                      |
| `__GNUC_PATCHLEVEL__`                                        | Patch number                       |
| `__clang__`                                                  | Clang compiler                     |
| `__clang_major__`, `__clang_minor__`, `__clang_patchlevel__` | Clang versions                     |

### MSVC

| Macro           | Meaning                                    |
| --------------- | ------------------------------------------ |
| `_MSC_VER`      | MSVC version (e.g., 1930 = VS2022)         |
| `_MSC_FULL_VER` | Full version                               |
| `_MSVC_LANG`    | Actual C++ standard mode (`201703L`, etc.) |

### Intel

| Macro              | Meaning                |
| ------------------ | ---------------------- |
| `__INTEL_COMPILER` | Intel compiler version |

---

# 🟨 **5. Platform / OS Macros**

| Macro       | Platform                 |
| ----------- | ------------------------ |
| `_WIN32`    | Windows (both 32/64 bit) |
| `_WIN64`    | Windows 64-bit           |
| `__linux__` | Linux                    |
| `__unix__`  | POSIX/Unix               |
| `__APPLE__` | Apple systems            |
| `__MACH__`  | macOS/iOS kernel         |

---

# ✔️ If you'd like…

I can also generate:

📌 **A single combined table** (all macros sorted alphabetically)
📌 **A header file template** that checks for features safely
📌 **A macro-based portable compatibility layer** for your project

Just tell me!

