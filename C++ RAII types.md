A **well-designed RAII type** in C++ is a class that **owns a resource and manages its lifetime automatically**:

- **Acquisition happens in the constructor**
    
- **Release happens in the destructor**
    
- Copying/moving rules are clearly defined
    
- The object remains valid throughout its lifetime
    

RAII stands for **Resource Acquisition Is Initialization**.

Instead of manually doing:

```cpp
Resource* r = acquire();

use(r);

release(r);
```

you wrap the resource:

```cpp
{
    ResourceHandle r;  // acquires resource
    use(r);
}                       // automatically releases resource
```

---

## Examples of well-designed RAII types in the C++ standard library

### 1. `std::string`

Manages dynamically allocated character memory.

```cpp
{
    std::string name = "Alice";

} // memory automatically released
```

You do not write:

```cpp
delete[] name;
```

because `std::string` owns its memory.

---

### 2. `std::vector`

Manages a dynamic array.

```cpp
{
    std::vector<int> numbers;

    numbers.push_back(10);
    numbers.push_back(20);

} // memory automatically freed
```

Internally:

```
vector object
    |
    v
+----------------+
| 10 | 20 | ...  |
+----------------+
```

The destructor releases the allocation.

---

### 3. `std::unique_ptr`

Represents **exclusive ownership**.

```cpp
{
    auto ptr = std::make_unique<int>(42);

    std::cout << *ptr;
}
```

When `ptr` goes out of scope:

```
unique_ptr destructor
        |
        v
    delete int
```

Important properties:

- Cannot be copied
    
- Can be moved
    

```cpp
auto p1 = std::make_unique<int>(10);

auto p2 = std::move(p1); // ownership transferred
```

---

### 4. `std::shared_ptr`

Represents **shared ownership**.

```cpp
{
    auto p1 = std::make_shared<int>(100);

    {
        auto p2 = p1;
    } // reference count decreases

} // last owner destroys object
```

Internally:

```
        shared_ptr
             |
             v
      +--------------+
      | object       |
      | count = 2    |
      +--------------+
```

When count reaches zero, the object is deleted.

---

### 5. `std::lock_guard`

Manages mutex locking.

Without RAII:

```cpp
mutex.lock();

doWork();

mutex.unlock();
```

Problem:

```cpp
mutex.lock();

throw Exception(); // unlock never happens
```

With RAII:

```cpp
{
    std::lock_guard<std::mutex> lock(mutex);

    doWork();

} // unlock automatically
```

The destructor releases the lock.

---

### 6. `std::unique_lock`

A more flexible mutex RAII wrapper.

Example:

```cpp
std::unique_lock<std::mutex> lock(mutex);

condition.wait(lock);
```

It supports:

- deferred locking
    
- unlocking/relocking
    
- moving ownership
    

---

### 7. `std::fstream`

Manages file handles.

```cpp
{
    std::ofstream file("data.txt");

    file << "Hello";

} // file automatically closed
```

No:

```cpp
fclose(file);
```

needed.

---

### 8. `std::filesystem::path`

Manages filesystem paths safely.

```cpp
std::filesystem::path p = "/home/user/file.txt";
```

It handles path representation instead of raw C strings.

---

## User-defined RAII example

Suppose you need to manage a database connection:

### Bad:

```cpp
DatabaseConnection* db = connect();

query(db);

disconnect(db);
```

A forgotten `disconnect()` leaks resources.

---

### RAII version:

```cpp
class DatabaseConnection {
public:
    DatabaseConnection() {
        connect();
    }

    ~DatabaseConnection() {
        disconnect();
    }

    void query();
};
```

Usage:

```cpp
{
    DatabaseConnection db;

    db.query();

} // disconnect() automatically called
```

---

## Characteristics of a good RAII type

A good RAII class usually has:

### 1. Clear ownership

Who owns the resource?

Example:

```cpp
std::unique_ptr<File>
```

means:

> "This object owns the file."

---

### 2. Exception safety

Resources are released even if exceptions occur.

```cpp
{
    std::vector<int> v;

    riskyFunction();

} // destructor still runs
```

---

### 3. Correct move semantics

Many RAII objects are movable:

```cpp
std::unique_ptr<int> a;

auto b = std::move(a);
```

Ownership transfers.

---

### 4. No manual cleanup required

Bad API:

```cpp
open();
close();
lock();
unlock();
allocate();
free();
```

Better:

```cpp
File f;
std::lock_guard lock(mutex);
std::vector<int> data;
```

The lifetime of the object controls the resource.

---

## Common RAII types you should know

|Resource|RAII type|
|---|---|
|Dynamic memory|`std::unique_ptr`, `std::shared_ptr`|
|Arrays|`std::vector`, `std::array`|
|Strings|`std::string`|
|Files|`std::fstream`|
|Mutex locks|`std::lock_guard`, `std::unique_lock`|
|Dynamic buffers|`std::vector`, `std::string`|
|Threads|`std::thread`, `std::jthread`|
|Time resources|`std::chrono` types|
|Filesystem paths|`std::filesystem::path`|

---

A good modern C++ design often looks like:

```cpp
class MyClass {
    std::string name;
    std::vector<int> data;
    std::unique_ptr<Resource> resource;
    std::mutex mutex;
};
```

No destructor. No copy/move functions. No manual cleanup.

That is the **Rule of Zero** in practice.