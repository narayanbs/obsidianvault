Good C++ error handling is about making failures **explicit, predictable, safe, and easy to diagnose**. The best approach depends on the type of error (recoverable vs. programmer bug vs. system failure), but these practices are widely recommended.

## 1. Prefer RAII for resource safety

Use constructors/destructors and standard library types so resources are automatically released.

**Avoid:**

```cpp
FILE* file = fopen("data.txt", "r");

if (!file) {
    return -1;
}

// Work with file...

fclose(file);
```

A future return path or exception could skip `fclose()`.

**Prefer:**

```cpp
#include <fstream>

std::ifstream file("data.txt");

if (!file) {
    throw std::runtime_error("Failed to open file");
}
```

Or use smart pointers:

```cpp
auto buffer = std::make_unique<char[]>(1024);
```

RAII prevents leaks even when exceptions occur.

---

## 2. Use exceptions for exceptional failures

Exceptions are appropriate when a function cannot complete its normal contract.

Example:

```cpp
double divide(double a, double b)
{
    if (b == 0.0) {
        throw std::invalid_argument("Division by zero");
    }

    return a / b;
}
```

Caller:

```cpp
try {
    auto result = divide(10, 0);
}
catch (const std::invalid_argument& e) {
    std::cerr << e.what();
}
```

Good exception candidates:

- File open failures
    
- Network failures
    
- Memory allocation failures
    
- Invalid states
    
- Resource acquisition failures
    

Avoid using exceptions for normal control flow.

---

## 3. Throw by value, catch by reference

Correct:

```cpp
throw std::runtime_error("Something failed");
```

Catch:

```cpp
catch (const std::exception& e)
{
    std::cerr << e.what();
}
```

Avoid:

```cpp
throw new std::runtime_error("error"); // Wrong
```

and:

```cpp
catch (std::exception e) // Copies exception
{
}
```

---

## 4. Use the standard exception hierarchy

Prefer standard exceptions when possible:

|Exception|Use case|
|---|---|
|`std::invalid_argument`|Bad function input|
|`std::out_of_range`|Index/key outside valid range|
|`std::runtime_error`|Runtime failures|
|`std::logic_error`|Programming mistakes|
|`std::system_error`|OS/library failures|
|`std::bad_alloc`|Memory allocation failure|

Example:

```cpp
std::string getUser(int id)
{
    if (id < 0)
        throw std::invalid_argument("Negative user id");

    if (!exists(id))
        throw std::runtime_error("User database unavailable");

    return loadUser(id);
}
```

---

## 5. Create custom exceptions for domain errors

For application-specific failures:

```cpp
#include <stdexcept>

class DatabaseException : public std::runtime_error
{
public:
    using std::runtime_error::runtime_error;
};
```

Usage:

```cpp
throw DatabaseException("Connection timeout");
```

Catch specifically:

```cpp
catch (const DatabaseException& e)
{
    logError(e.what());
}
```

---

## 6. Do not swallow exceptions

Bad:

```cpp
try {
    process();
}
catch (...) {
    // ignore
}
```

This hides bugs.

Better:

```cpp
try {
    process();
}
catch (const std::exception& e) {
    log(e.what());
    throw; // preserve failure
}
```

---

## 7. Catch exceptions at meaningful boundaries

Avoid catching everywhere.

A common architecture:

```
main()
 |
 |-- application layer
 |
 |-- business logic
 |
 |-- library functions (throw)
```

Libraries throw:

```cpp
throw NetworkError("Connection failed");
```

The application boundary handles:

```cpp
int main()
{
    try {
        runApplication();
    }
    catch (const std::exception& e) {
        std::cerr << "Fatal error: "
                  << e.what()
                  << '\n';
        return EXIT_FAILURE;
    }
}
```

---

## 8. Use `std::expected` when failure is part of normal flow (C++23)

For operations where failure is expected, exceptions may be unnecessary.

Example:

```cpp
#include <expected>

std::expected<int, std::string> parseNumber(std::string input)
{
    try {
        return std::stoi(input);
    }
    catch (...) {
        return std::unexpected("Invalid number");
    }
}
```

Usage:

```cpp
auto result = parseNumber("abc");

if (!result) {
    std::cout << result.error();
}
```

Good for:

- Parsing
    
- Validation
    
- User input
    
- Optional operations
    

---

## 9. Use error codes only when appropriate

Some environments prefer explicit error handling:

```cpp
enum class ErrorCode
{
    Success,
    FileMissing,
    PermissionDenied
};

ErrorCode loadConfig(Config& cfg);
```

Useful for:

- Embedded systems
    
- Real-time systems
    
- Performance-critical code
    
- C APIs
    

Less ideal for complex applications because errors can be ignored.

---

## 10. Provide useful error messages

Bad:

```
Error
```

Better:

```
Failed to open configuration file '/etc/app/config.json': permission denied
```

Include:

- What operation failed
    
- Relevant object/value
    
- Context
    
- Original error if available
    

Example:

```cpp
throw std::runtime_error(
    "Unable to load user profile: id=" +
    std::to_string(userId)
);
```

---

## 11. Preserve exception context

Use nested exceptions:

```cpp
#include <exception>

try {
    readDatabase();
}
catch (...) {
    std::throw_with_nested(
        std::runtime_error("Loading configuration failed")
    );
}
```

Later:

```cpp
catch (const std::exception& e)
{
    std::cerr << e.what();
}
```

This preserves the root cause.

---

## 12. Mark functions that do not throw

Use `noexcept` when a function guarantees no exceptions:

```cpp
void cleanup() noexcept
{
    // Cannot throw
}
```

Good candidates:

- Destructors
    
- Move constructors
    
- Swap functions
    

Avoid:

```cpp
void process() noexcept
{
    throw std::runtime_error("error"); // terminates program
}
```

---

## 13. Never throw exceptions from destructors

Bad:

```cpp
class File
{
    ~File()
    {
        throw std::runtime_error("close failed");
    }
};
```

During stack unwinding, this can terminate the program.

Prefer:

```cpp
~File() noexcept
{
    close();
}
```

Log failures if needed.

---

## 14. Validate inputs early

Fail close to the source:

```cpp
void setAge(int age)
{
    if (age < 0 || age > 150)
        throw std::out_of_range("Invalid age");

    this->age = age;
}
```

Avoid allowing invalid state.

---

## 15. Use assertions for programmer errors

Assertions are for conditions that should never happen:

```cpp
#include <cassert>

void process(Item* item)
{
    assert(item != nullptr);
}
```

Use exceptions for runtime problems:

```cpp
if (!item)
    throw std::invalid_argument("item is null");
```

---

## 16. Log once, handle once

A common mistake:

```
Function A logs error
    ↓
Function B logs error
    ↓
main logs error
```

Result:

```
ERROR: File missing
ERROR: Loading failed
ERROR: Application failed
```

Prefer logging at the boundary where recovery or termination happens.

---

## Recommended modern C++ strategy

A practical approach:

|Situation|Recommended tool|
|---|---|
|Invalid programmer state|`assert()`|
|Recoverable expected failure|`std::expected`|
|Unexpected runtime failure|Exceptions|
|Resource management|RAII|
|System/API errors|`std::error_code`|
|Application termination|Catch at top level|

A typical modern C++ application often follows:

```
RAII everywhere
        ↓
std::expected for expected failures
        ↓
Exceptions for exceptional failures
        ↓
Catch/log at application boundaries
```

This gives predictable behavior without spreading error-handling code throughout the entire codebase.