The **C++ Standard Template Library (STL)** is a collection of reusable classes and functions that provide common data structures and algorithms. The most essential STL components are:

1. **Containers** – Store data.
2. **Iterators** – Traverse containers.
3. **Algorithms** – Perform operations like sorting and searching.
4. **Function Objects (Functors)** – Callable objects used by algorithms.
5. **Utilities** – Helper classes like `pair`, `tuple`, and smart pointers.

---

# 1. Sequence Containers

These store elements in a linear order.

## `vector`

A dynamic array with fast random access.

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {10, 20, 30};

    v.push_back(40);

    for (int x : v)
        cout << x << " ";
}
```

Output

```
10 20 30 40
```

**Common functions**

```cpp
v.push_back(5);
v.pop_back();
v.size();
v.empty();
v.front();
v.back();
v[2];
v.at(2);
```

---

## `deque`

Double-ended queue.

```cpp
deque<int> d;

d.push_front(5);
d.push_back(10);
d.push_back(20);

for(int x : d)
    cout << x << " ";
```

Output

```
5 10 20
```

---

## `list`

Doubly linked list.

```cpp
list<int> l = {1,2,3};

l.push_front(0);
l.push_back(4);

for(int x : l)
    cout << x << " ";
```

Output

```
0 1 2 3 4
```

Efficient insertion/deletion anywhere.

---

# 2. Associative Containers

Automatically keep elements sorted.

## `set`

Stores unique values.

```cpp
set<int> s;

s.insert(5);
s.insert(2);
s.insert(5);

for(int x : s)
    cout << x << " ";
```

Output

```
2 5
```

---

## `multiset`

Allows duplicates.

```cpp
multiset<int> ms;

ms.insert(5);
ms.insert(5);
ms.insert(2);

for(int x : ms)
    cout << x << " ";
```

Output

```
2 5 5
```

---

## `map`

Stores key-value pairs.

```cpp
map<string,int> age;

age["Alice"] = 20;
age["Bob"] = 25;

cout << age["Alice"];
```

Output

```
20
```

Traversal:

```cpp
for(auto p : age)
    cout << p.first << " " << p.second << endl;
```

Output

```
Alice 20
Bob 25
```

---

## `multimap`

Allows duplicate keys.

```cpp
multimap<int,string> mm;

mm.insert({1,"A"});
mm.insert({1,"B"});
```

---

# 3. Unordered Containers

Implemented using hash tables.

## `unordered_set`

```cpp
unordered_set<int> us;

us.insert(10);
us.insert(20);
us.insert(10);

for(int x : us)
    cout << x << " ";
```

Order is **not guaranteed**.

---

## `unordered_map`

```cpp
unordered_map<string,int> marks;

marks["Tom"] = 90;
marks["Jerry"] = 85;

cout << marks["Tom"];
```

---

# 4. Container Adaptors

These are built on top of other containers.

## `stack`

LIFO (Last In First Out)

```cpp
stack<int> st;

st.push(10);
st.push(20);
st.push(30);

cout << st.top() << endl;

st.pop();

cout << st.top();
```

Output

```
30
20
```

---

## `queue`

FIFO (First In First Out)

```cpp
queue<int> q;

q.push(1);
q.push(2);
q.push(3);

cout << q.front() << endl;

q.pop();

cout << q.front();
```

Output

```
1
2
```

---

## `priority_queue`

Max heap by default.

```cpp
priority_queue<int> pq;

pq.push(30);
pq.push(10);
pq.push(50);

cout << pq.top();
```

Output

```
50
```

Min heap:

```cpp
priority_queue<int,
               vector<int>,
               greater<int>> pq;
```

---

# 5. Iterators

Used to traverse containers.

```cpp
vector<int> v = {10,20,30};

for(auto it = v.begin(); it != v.end(); ++it)
    cout << *it << " ";
```

Output

```
10 20 30
```

Iterator functions

```cpp
begin()
end()
rbegin()
rend()
cbegin()
cend()
```

---

# 6. Algorithms

Include `<algorithm>`.

## Sort

```cpp
vector<int> v = {5,1,3,2};

sort(v.begin(), v.end());
```

Output

```
1 2 3 5
```

Descending

```cpp
sort(v.begin(), v.end(), greater<int>());
```

---

## Reverse

```cpp
reverse(v.begin(), v.end());
```

---

## Find

```cpp
auto it = find(v.begin(), v.end(), 3);

if(it != v.end())
    cout << "Found";
```

---

## Count

```cpp
vector<int> v = {1,2,2,3};

cout << count(v.begin(), v.end(), 2);
```

Output

```
2
```

---

## Binary Search

```cpp
sort(v.begin(), v.end());

if(binary_search(v.begin(), v.end(), 3))
    cout << "Found";
```

---

## Max and Min

```cpp
cout << *max_element(v.begin(), v.end());

cout << *min_element(v.begin(), v.end());
```

---

## Accumulate

```cpp
#include <numeric>

vector<int> v = {1,2,3,4};

cout << accumulate(v.begin(), v.end(), 0);
```

Output

```
10
```

---

# 7. Utility Classes

## `pair`

```cpp
pair<int,string> p = {1,"Apple"};

cout << p.first << endl;
cout << p.second;
```

Output

```
1
Apple
```

---

## `tuple`

```cpp
tuple<int,string,double> t(1,"Book",99.5);

cout << get<1>(t);
```

Output

```
Book
```

---

# 8. Function Objects (Functors)

```cpp
sort(v.begin(), v.end(), greater<int>());
```

`greater<int>()` is a predefined functor.

Custom functor:

```cpp
struct Compare {
    bool operator()(int a, int b) {
        return a > b;
    }
};

sort(v.begin(), v.end(), Compare());
```

---

# 9. Lambda Expressions

Often used instead of custom functors.

```cpp
sort(v.begin(), v.end(),
    [](int a, int b) {
        return a > b;
    });
```

---

# 10. Common STL Time Complexities

| Container        | Insert               | Delete       | Search       | Access       |
| ---------------- | -------------------- | ------------ | ------------ | ------------ |
| `vector`         | O(1) amortized (end) | O(1) end     | O(n)         | O(1)         |
| `deque`          | O(1) ends            | O(1) ends    | O(n)         | O(1)         |
| `list`           | O(1)                 | O(1)         | O(n)         | O(n)         |
| `set`            | O(log n)             | O(log n)     | O(log n)     | O(log n)     |
| `map`            | O(log n)             | O(log n)     | O(log n)     | O(log n)     |
| `unordered_set`  | O(1) average         | O(1) average | O(1) average | —            |
| `unordered_map`  | O(1) average         | O(1) average | O(1) average | O(1) average |
| `priority_queue` | O(log n) push/pop    | O(log n)     | O(1) top     | O(1) top     |

## STL Headers Cheat Sheet

| Header            | Main Classes/Functions                                                            |
| ----------------- | --------------------------------------------------------------------------------- |
| `<vector>`        | `vector`                                                                          |
| `<deque>`         | `deque`                                                                           |
| `<list>`          | `list`                                                                            |
| `<set>`           | `set`, `multiset`                                                                 |
| `<map>`           | `map`, `multimap`                                                                 |
| `<unordered_set>` | `unordered_set`                                                                   |
| `<unordered_map>` | `unordered_map`                                                                   |
| `<stack>`         | `stack`                                                                           |
| `<queue>`         | `queue`, `priority_queue`                                                         |
| `<algorithm>`     | `sort`, `find`, `reverse`, `count`, `binary_search`, `max_element`, `min_element` |
| `<numeric>`       | `accumulate`, `iota`                                                              |
| `<utility>`       | `pair`                                                                            |
| `<tuple>`         | `tuple`                                                                           |

### Which STL containers should you learn first?

For coding interviews, competitive programming, and day-to-day C++ development, focus on these in order:

1. `vector`
2. `pair`
3. `map` and `unordered_map`
4. `set` and `unordered_set`
5. `stack`
6. `queue`
7. `priority_queue`
8. Iterators and range-based `for`
9. Algorithms (`sort`, `find`, `count`, `lower_bound`, `upper_bound`, `binary_search`, `reverse`, `accumulate`)
10. Lambdas and custom comparators

These cover the vast majority of STL usage in practice.
