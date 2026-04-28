
## Good Suffix Heuristic 

#### Border Preprocessing

You’re looking at the **border preprocessing step** used in pattern-matching algorithms like the **good suffix rule** in string searching (e.g., Boyer–Moore). Let’s break it down in a simple way.

---

## 🔹 What is a “border” again?

A **border** of a string is a substring that is:

* a **prefix** (start of the string), and
* a **suffix** (end of the string),
* but **not the whole string** (that’s why it’s called *proper*).

Example:
For `"ccacc"`

* `"c"` ✅ (prefix & suffix)
* `"cc"` ✅
* `"cca"` ❌ (not a suffix)

---

## 🔹 What does `bpos[i]` mean?

Let’s say:

* `P` = pattern
* `m` = length of pattern

Then:

👉 `bpos[i]` stores the **starting index of the longest border** of the **suffix starting at index `i`**.

In simpler terms:

* Take the substring `P[i ... m-1]` (a suffix)
* Find its **longest border**
* Store **where that border starts in the pattern**

---

## 🔹 Why is `bpos[m] = m + 1`?

* `i = m` means you're looking at an **empty suffix**
* An empty string has **no border**
* So we set a dummy value: `m + 1` (just a convention to indicate “no border”)

---

## 🔹 Example to understand clearly

For the pattern **"ababa"** (length ( m = 5 )), let’s compute the `bpos` array step by step using your function.

### Final `bpos` values

Indexing from **0 to 5** (since the array is of size ( m+1 )):

```
bpos[0] = 2
bpos[1] = 3
bpos[2] = 4
bpos[3] = 5
bpos[4] = 5
bpos[5] = 6
```

### So the array is:

```
[2, 3, 4, 5, 5, 6]
```

## Quick Intuition

Python

```
def preprocess_strong_suffix(pat):
    m = len(pat)
    shift = [0] * (m + 1)
    bpos = [0] * (m + 1)
    i = m
    j = m + 1
    bpos[i] = j
    
    trace = []
    
    while i > 0:
        while j <= m and pat[i - 1] != pat[j - 1]:
            if shift[j] == 0:
                shift[j] = j - i
            j = bpos[j]
        i -= 1
        j -= 1
        bpos[i] = j
        trace.append((i, j, bpos[:]))
        
    return bpos, shift, trace

pat = "ababa"
bpos, shift, trace = preprocess_strong_suffix(pat)
print(f"Pattern: {pat}")
print(f"bpos: {bpos}")
print(f"shift: {shift}")


```

Code output

```
Pattern: ababa
bpos: [2, 3, 4, 5, 5, 6]
shift: [0, 0, 0, 0, 0, 1]

```

To find the `bpos` array for the string **"ababa"**, we need to trace the `preprocess_strong_suffix` function. This function is a core part of the Boyer-Moore string searching algorithm, specifically for the **Good Suffix Heuristic**.

The `bpos` (Border Position) array stores the starting position of the widest border of each suffix in the pattern.

### 1. Initialization

- **Pattern ($pat$):** `a b a b a`
    
- **Indices:** `0 1 2 3 4`
    
- **Length ($m$):** 5
    
- **Initial values:** `i = 5`, `j = 6`, `bpos[5] = 6`.
    

### 2. Step-by-Step Trace

The loop runs while `i > 0`.

|**Iteration**|**i (start)**|**j (start)**|**Logic / Inner While Loop**|**i (end)**|**j (end)**|**bpos[i]**|
|---|---|---|---|---|---|---|
|**Start**|5|6|-|-|-|`bpos[5] = 6`|
|**1**|5|6|`j > m`, skip inner while.|4|5|`bpos[4] = 5`|
|**2**|4|5|`pat[3]('b') != pat[4]('a')`. $j$ becomes `bpos[5] = 6`.|3|5|`bpos[3] = 5`|
|**3**|3|5|`pat[2]('a') == pat[4]('a')`. Match! Inner while exit.|2|4|`bpos[2] = 4`|
|**4**|2|4|`pat[1]('b') == pat[3]('b')`. Match! Inner while exit.|1|3|`bpos[1] = 3`|
|**5**|1|3|`pat[0]('a') == pat[2]('a')`. Match! Inner while exit.|0|2|`bpos[0] = 2`|

### 3. Final `bpos` Array

For the pattern **"ababa"**, the resulting `bpos` array is:

|**Index**|**0**|**1**|**2**|**3**|**4**|**5**|
|---|---|---|---|---|---|---|
|**Value**|**2**|**3**|**4**|**5**|**5**|**6**|

### Summary of what these values represent:

The `bpos[i]` value indicates where the suffix starting at `i` repeats itself as a border.

- **bpos[0] = 2**: The suffix starting at index 0 ("ababa") has a border starting at index 2 ("aba").
    
- **bpos[1] = 3**: The suffix starting at index 1 ("baba") has a border starting at index 3 ("ba").
    
- **bpos[2] = 4**: The suffix starting at index 2 ("aba") has a border starting at index 4 ("a").
    
- **bpos[4] = 5**: The suffix starting at index 4 ("a") has no border (represented by index $m=5$).
