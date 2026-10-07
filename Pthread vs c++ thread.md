Since you already know pthreads, here's a **pthreads → modern C++ mental mapping** rather than a beginner explanation.

## 1\. Creating a thread

 With pthreads:

```
pthread_t t;
pthread_create(&t, nullptr, worker, arg);
```

 With C++:

```
#include <thread>

void worker(int x) {
    // ...
}

std::thread t(worker, 42);
```

 The arguments after the callable are passed to the thread function.

 You can also use lambdas:

```
std::thread t([] {
    // runs in another thread
});
```

 ### Joining

```
t.join();
```

 Equivalent conceptually to:

```
pthread_join(t, nullptr);
```

 You **must** deal with the thread before its `std::thread` object is destroyed.

---

## 2\. `detach()`

```
std::thread t(worker);
t.detach();
```

 This is roughly equivalent to:

```
pthread_detach(thread);
```

 The thread continues independently.

 **Be careful:** detached threads make lifetime management much harder.

 In modern C++, prefer `join()` unless you have a specific reason to detach.

---

## 3\. `std::jthread` — usually better in C++20+

 If you're using C++20, learn this too:

```
#include <thread>

std::jthread t([] {
    // work
});
```

 Unlike `std::thread`, `jthread` automatically joins when it goes out of scope.

 That's a huge improvement for RAII.

```
void foo()
{
    std::jthread t([] {
        do_work();
    });

    // ...
} // t automatically joins here
```

 For new C++20 code, I'd generally reach for **`std::jthread` before `std::thread`** when I actually need a directly managed thread.

---

## 4\. Passing references

 This is one place where C++'s type system matters.

 This **doesn't** necessarily do what you want:

```
int x = 42;

std::thread t([](int& x) {
    x++;
}, x);
```

 Use `std::ref`:

```
std::thread t([](int& x) {
    x++;
}, std::ref(x));
```

 Or capture by reference with a lambda:

```
std::thread t([&x] {
    x++;
});
```

 Think of `std::ref()` as roughly helping you say:

 > "Don't copy this argument; pass a reference wrapper."

---

## 5\. Mutexes

 Pthreads:

```
pthread_mutex_t mutex;
pthread_mutex_lock(&mutex);

// critical section

pthread_mutex_unlock(&mutex);
```

 C++:

```
std::mutex mutex;

mutex.lock();

// critical section

mutex.unlock();
```

 But **don't normally manually unlock it**.

 Use RAII:

```
std::lock_guard<std::mutex> lock(mutex);

// critical section
```

 When `lock` leaves scope, the mutex is automatically unlocked.

 This is one of the biggest differences in style:

```
pthread:
    lock()
    ...
    unlock()

C++:
    construct guard
    ...
    guard destructs → unlock()
```

---

## 6\. `unique_lock`

 You'll use this a lot with condition variables:

```
std::unique_lock<std::mutex> lock(mutex);

cv.wait(lock);
```

 Compared with `lock_guard`, `unique_lock` is more flexible:

```
std::unique_lock lock(mutex);

lock.unlock();

// do stuff without mutex

lock.lock();
```

 C++ can also infer the mutex type:

```
std::unique_lock lock(mutex);
```

 if you're using C++17+.

---

## 7\. Condition variables

 Pthreads:

```
pthread_cond_wait(&cond, &mutex);
pthread_cond_signal(&cond);
```

 C++:

```
std::condition_variable cv;

std::unique_lock lock(mutex);

cv.wait(lock);
```

 Notify:

```
cv.notify_one();
```

 or:

```
cv.notify_all();
```

 But the **important C++ pattern** is:

```
cv.wait(lock, [] {
    return !queue.empty();
});
```

 This is the idiomatic way.

 A classic producer/consumer example:

```
std::mutex mutex;
std::condition_variable cv;
std::queue<int> queue;

void consumer()
{
    while (true) {
        std::unique_lock lock(mutex);

        cv.wait(lock, [] {
            return !queue.empty();
        });

        int value = queue.front();
        queue.pop();

        lock.unlock();

        process(value);
    }
}

void producer(int value)
{
    {
        std::lock_guard lock(mutex);
        queue.push(value);
    }

    cv.notify_one();
}
```

 Notice that we only hold the mutex while touching the queue.

---

## 8\. Thread IDs

 Pthreads:

```
pthread_self();
```

 C++:

```
std::this_thread::get_id();
```

 For example:

```
std::cout << std::this_thread::get_id();
```

 And:

```
t.get_id();
```

 gets the ID of a particular `std::thread`.

---

## 9\. Sleeping

 Instead of:

```
sleep(1);
```

 use:

```
std::this_thread::sleep_for(std::chrono::seconds(1));
```

 Or:

```
using namespace std::chrono_literals;

std::this_thread::sleep_for(100ms);
```

 The chrono integration is much nicer than POSIX time APIs.

---

 ## 10\. Atomics

 You don't need a mutex for every shared variable.

```
std::atomic<int> counter{0};

void worker()
{
    counter.fetch_add(1);
}
```

 Or simply:

```
counter++;
```

 for an atomic integer.

 This is where I'd strongly recommend you spend some time if you're coming from pthreads: **understanding the C++ memory model and atomic memory ordering**.

 For example:

```
std::atomic<bool> ready{false};
```

 is fundamentally different from:

```
bool ready = false;
```

 when accessed concurrently.

---

 ## 11\. Exceptions

 A very important gotcha:

```
std::thread t([] {
    throw std::runtime_error("oops");
});
```

 An exception escaping the thread function results in `std::terminate()`.

 You need to catch it inside the thread if appropriate:

```
std::thread t([] {
    try {
        work();
    }
    catch (...) {
        // handle / communicate exception
    }
});
```

 For higher-level async work, `std::future` can propagate exceptions back to the caller.

---

 ## 12\. The biggest mental shift

 With pthreads, you might naturally think:

 > "I need to create/manage threads."

 Modern C++ increasingly encourages:

 > **"I have work that needs to execute concurrently."**

 For example, instead of manually creating 100 threads:

```
for (int i = 0; i < 100; ++i)
    std::thread(...);
```

 you'll eventually want concepts like:

```
tasks
    ↓
thread pool
    ↓
workers
```

 The standard library doesn't give you a full general-purpose thread pool in the traditional C++20/23 library, so production C++ code often uses a library or a custom executor/thread pool.

---

 ## Your pthread → C++ cheat sheet

```
pthread_create()          → std::thread(...)
pthread_join()            → thread.join()
pthread_detach()          → thread.detach()

pthread_mutex_t           → std::mutex
pthread_mutex_lock()      → lock_guard / unique_lock
pthread_mutex_unlock()    → RAII

pthread_cond_t            → condition_variable
pthread_cond_wait()       → cv.wait()
pthread_cond_signal()     → cv.notify_one()
pthread_cond_broadcast()  → cv.notify_all()

pthread_self()            → this_thread::get_id()

sleep()                   → this_thread::sleep_for()

pthread_once()            → call_once()

pthread_rwlock_t          → shared_mutex

atomic operations         → std::atomic<T>

pthread cancellation      → stop_token / cooperative cancellation

manual lifetime          → RAII
```

 ### If you only remember 5 things

```
std::thread
std::jthread

std::mutex
std::lock_guard / std::unique_lock
std::condition_variable
```

 And then learn:

```
std::atomic
std::future
std::promise
std::stop_token
```

 Given that you already know pthreads, **the next thing I'd learn isn't `std::thread` itself — it's C++ RAII + the C++ memory model/atomics**. That's where the really important differences start.