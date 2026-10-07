~~~c++

#include <chrono>
#include <iostream>
#include <memory>  // For std::shared_ptr
#include <mutex>
#include <thread>
#include <vector>

std::mutex mtx;

// Pass by shared_ptr instead of raw reference
void worker(std::shared_ptr<int> x_ptr) {
  std::lock_guard<std::mutex> lock(mtx);
  std::cout << "Incrementing " << *x_ptr << "\n";
  (*x_ptr)++;
  std::this_thread::sleep_for(std::chrono::milliseconds(200));
}

int main() {
  // x is now allocated on the heap and managed safely
  auto x = std::make_shared<int>(20);
  std::vector<std::thread> threads;

  for (unsigned int i = 0; i < 5; i++) {
    // Each thread gets its own shared copy of the pointer, keeping the memory alive
    threads.push_back(std::thread(worker, x));
  }

  for (auto &t : threads) {
    t.join();
  }

  std::cout << "X is " << *x << std::endl;
  return 0;
}
~~~

