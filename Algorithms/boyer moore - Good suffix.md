Let’s do a **complete worked example** so you can see exactly how `bpos[]` and `shift[]` are built and used in the good suffix rule of the Boyer–Moore string search algorithm.

---

# 🔹 Example Pattern

```
P = "ABAB"
m = 4
Index:  0 1 2 3
        A B A B
```

We will compute:

- `bpos[0..m]`
    
- `shift[0..m]`
    

---

# ✅ Phase 1: Build `bpos[]` and partial `shift[]`

Initialize:

```
i = m = 4
j = m + 1 = 5

bpos[4] = 5
shift[] = [0, 0, 0, 0, 0]
```

---

## 🔁 Iteration steps

### Step 1:

```
i = 4, j = 5
→ decrement: i=3, j=4
bpos[3] = 4
```

---

### Step 2:

Compare:

```
P[i-1] = P[2] = 'A'
P[j-1] = P[3] = 'B' → mismatch
```

So:

```
shift[4] = j - i = 4 - 3 = 1
j = bpos[4] = 5
```

Now:

```
j > m → stop inner loop
```

Then:

```
i=2, j=4
bpos[2] = 4
```

---

### Step 3:

Compare:

```
P[1] = 'B'
P[3] = 'B' → match
```

So:

```
i=1, j=3
bpos[1] = 3
```

---

### Step 4:

Compare:

```
P[0] = 'A'
P[2] = 'A' → match
```

So:

```
i=0, j=2
bpos[0] = 2
```

---

## ✅ After Phase 1

### `bpos[]`:

```
Index:   0 1 2 3 4
bpos:    2 3 4 4 5
```

### `shift[]` (partial):

```
shift:   0 0 0 0 1
```

---

# ✅ Phase 2: Fill remaining `shift[]`

Initialize:

```
j = bpos[0] = 2
```

---

## 🔁 Fill loop

### i = 0:

```
shift[0] = 2
```

### i = 1:

```
shift[1] = 2
```

### i = 2:

```
i == j → update j = bpos[2] = 4
shift[2] = 2
```

### i = 3:

```
shift[3] = 4
```

### i = 4:

```
i == j → j = bpos[4] = 5
(shift[4] already = 1)
```

---

# ✅ Final Arrays

### 🔹 `bpos[]`

```
Index:   0 1 2 3 4
bpos:    2 3 4 4 5
```

---

### 🔹 `shift[]`

```
Index:   0 1 2 3 4
shift:   2 2 2 4 1
```

---

# 🔹 How to interpret this

- If mismatch happens at position `i`, use:
    

```
shift[i+1]
```

### Example:

- Mismatch at index `2` → shift = `shift[3] = 4`
    

---

# 🔹 Intuition from this example

Pattern: `"ABAB"`

- Suffix `"AB"` appears twice → we reuse it
    
- That’s why many shifts are small (`2`)
    
- When no reuse → fallback to prefix → bigger shift (`4`)
    

---

# 🔥 Key takeaway

- `bpos[]` = **where to fallback (structure)**
    
- `shift[]` = **how far to jump (action)**
    

---

If you want, I can next:

- Do a **full pattern matching run using this table**, or
    
- Show a **trick to remember the algorithm in exams** (super useful)
------------------------
Great — let’s **use the `bpos[]` and `shift[]` we built** to actually perform pattern matching. This is where the good suffix rule of the Boyer–Moore string search algorithm really clicks.

---

# 🔹 Pattern and Text

### Pattern:

```
P = "ABAB"
m = 4
```

### Precomputed:

```
shift = [2, 2, 2, 4, 1]
```

---

### Text:

```
T = "ABACABAB"
Index:  0 1 2 3 4 5 6 7
        A B A C A B A B
```

---

# 🔹 Matching Process (Right to Left)

We align pattern with text and compare **from rightmost character**.

---

## ✅ Step 1: Align at position s = 0

```
T: A B A C A B A B
P: A B A B
      ↑
```

Compare from right:

- P[3] = B vs T[3] = C → ❌ mismatch at i = 3
    

👉 Shift:

```
shift[i+1] = shift[4] = 1
```

✅ Move pattern by **1**

---

## ✅ Step 2: s = 1

```
T: A B A C A B A B
P:   A B A B
        ↑
```

Compare:

- P[3] = B vs T[4] = A → ❌ mismatch at i = 3
    

👉 Shift:

```
shift[4] = 1
```

✅ Move by **1**

---

## ✅ Step 3: s = 2

```
T: A B A C A B A B
P:     A B A B
          ↑
```

Compare:

- P[3] = B vs T[5] = B → ✅
    
- P[2] = A vs T[4] = A → ✅
    
- P[1] = B vs T[3] = C → ❌ mismatch at i = 1
    

👉 Shift:

```
shift[i+1] = shift[2] = 2
```

✅ Move by **2**

---

## ✅ Step 4: s = 4

```
T: A B A C A B A B
P:         A B A B
            ↑
```

Compare:

- P[3] = B vs T[7] = B → ✅
    
- P[2] = A vs T[6] = A → ✅
    
- P[1] = B vs T[5] = B → ✅
    
- P[0] = A vs T[4] = A → ✅
    

🎉 **Full match found at index 4**

---

👉 After full match:

```
shift[0] = 2
```

Move by **2**

---

## ✅ Step 5: s = 6

Out of bounds → stop

---

# 🔹 Final Result

Pattern `"ABAB"` found at:

```
Index = 4
```

---

# 🔥 What you should notice

### 1. We never moved character-by-character (like naive)

- Sometimes shift = 1
    
- Sometimes shift = 2
    

👉 That’s the power of **good suffix rule**

---

### 2. Key moment:

At **Step 3**:

- We matched `"AB"` suffix
    
- Instead of restarting → we reused it
    
- Jumped by `2`
    

---

### 3. Formula you must remember

When mismatch at index `i`:

```
Shift = shift[i + 1]
```

When full match:

```
Shift = shift[0]
```

---

# 🧠 Exam Trick (very useful)

If you forget everything, remember:

👉 **"Mismatch at i → look one step right (i+1) in shift[]"**

---

# 🚀 Final Summary

- `bpos[]` → builds fallback structure
    
- `shift[]` → gives jump distances
    
- Matching goes **right → left**
    
- On mismatch:
    
    ```
    shift[i+1]
    ```
    
- On match:
    
    ```
    shift[0]
    ```
    

---

If you want, next I can:

- Combine **bad character + good suffix** (full Boyer–Moore), or
    
- Give a **super short cheat sheet** for exams 👍

-----------------------
----------------------

In the **Boyer–Moore string search algorithm**, the **`shift` array** (often called the _good suffix shift table_) tells you:

> **How far you can safely move the pattern to the right when a mismatch happens at a given position.**

---

### 🔹 What your `shift` array means

For your pattern **ABAB** (length = 4), you have:

- `shift = [2, 2, 2, 4, 1]`
    

This array is typically indexed from **0 to m (pattern length)**, where:

- `shift[i]` = how much to shift when a mismatch happens at position **i−1** in the pattern (or after matching suffix starting at i)
    

---

### 🔹 Interpreting each value

Let’s map it clearly:

|Index `i`|Situation|`shift[i]` meaning|
|---|---|---|
|0|mismatch at first char|shift by 2|
|1|mismatch after matching suffix of length 1|shift by 2|
|2|mismatch after matching suffix of length 2|shift by 2|
|3|mismatch after matching suffix of length 3|shift by 4|
|4|full match|shift by 1|

---

### 🔹 Intuition (important)

The shift value answers:

👉 _“Given what I matched so far (a suffix), how far can I slide the pattern so that another possible match lines up?”_

It uses two ideas:

1. **Good suffix rule**:
    
    - If a suffix matched, try to align another occurrence of that suffix in the pattern.
        
2. **Border fallback (via `bpos`)**:
    
    - If no such occurrence exists, align the longest prefix that matches a suffix.
        

---

### 🔹 Example with your pattern `ABAB`

Suppose you're matching from right to left:

- If you matched `"AB"` at the end but got a mismatch before it  
    → shift by `shift[2] = 2`
    

Why?  
Because `"AB"` appears earlier in the pattern, so you align that.

---

### 🔹 Special cases

- `shift[4] = 1`  
    → after a **full match**, move by 1 to find overlapping matches
    
- `shift[3] = 4`  
    → large shift because no good suffix match exists, so fallback to border rule
    

---

### 🔹 In one sentence

👉 The **`shift` array tells you how far to jump the pattern after a mismatch, based on the suffix you already matched, so you skip unnecessary comparisons.**

---



---

## 🔹 Key idea (fix the interpretation)

In **Boyer–Moore string search algorithm** (good suffix rule):

> `shift[i]` is **not really about a mismatch at index `i-1` in a literal sense**.

Instead, think of it as:

> **`shift[i]` = how far to shift when a suffix of length `m - i` has matched**

(where `m` = pattern length)

---

## 🔹 So what is `shift[0]`?

For your pattern **ABAB** (m = 4):

- `shift[0]` corresponds to:
    
    - matched suffix length = `m - 0 = 4`
        
    - 👉 meaning **the entire pattern matched**
        

So:

> 🔸 `shift[0]` = shift after a **full match**

NOT mismatch at -1.

---

## 🔹 Why does “i−1” show up in some explanations?

Some books say:

> “shift[i] is used when mismatch occurs at position i−1”

This works **only if you imagine the matching process backwards**:

- You compare from right → left
    
- If mismatch happens _before the suffix starting at i_, then suffix `[i..m-1]` already matched
    

So:

- mismatch at `i-1` ⇒ suffix starting at `i` matched
    

---

## 🔹 Clean mapping (correct mental model)

|i|What actually happened|Use shift[i]|
|---|---|---|
|0|full match|shift[0]|
|1|matched suffix of length 3|shift[1]|
|2|matched suffix of length 2|shift[2]|
|3|matched suffix of length 1|shift[3]|
|4|matched suffix of length 0 (immediate mismatch at last char)|shift[4]|

---

## 🔹 So your confusion resolved

You said:

> “shift[0] means mismatch at -1?”

❌ Not really.

✔️ Correct interpretation:

> **`shift[0]` is used after a full match of the pattern.**

---

## 🔹 Why this matters

This interpretation:

- avoids weird negative indices
    
- aligns perfectly with how suffixes are actually used
    
- makes debugging implementations MUCH easier
    

---

If you want, I can show you exactly how your  
`bpos = [2, 3, 4, 4, 5]` produced that `shift` array step-by-step — that’s the missing piece most explanations skip.


Great — let’s make this _click_ by actually running the algorithm.

We’ll use:

- Pattern **P = "ABAB"** (m = 4)
    
- `shift = [2, 2, 2, 4, 1]`
    
- `bpos = [2, 3, 4, 4, 5]`
    

And a sample text:

> **T = "ABACABAB"**

---

# 🔹 Step-by-step matching (right → left)

We align the pattern and compare from the **rightmost character**.

---

## ✅ Alignment 1 (shift = 0)

```
Text:    A  B  A  C  A  B  A  B
Pattern: A  B  A  B
          ↑ alignment starts at index 0
```

Compare from right:

- P[3] = B vs T[3] = C ❌ mismatch
    

👉 This is a mismatch at **pattern index 3**

Now determine shift:

- matched suffix length = 0
    
- so we use **shift[4] = 1**
    

✅ **Shift pattern by 1**

---

## ✅ Alignment 2 (shift = 1)

```
Text:    A  B  A  C  A  B  A  B
Pattern:    A  B  A  B
             ↑ starts at index 1
```

Compare:

- P[3] = B vs T[4] = A ❌ mismatch
    

Again:

- mismatch at last char → suffix length = 0
    
- use **shift[4] = 1**
    

✅ **Shift by 1**

---

## ✅ Alignment 3 (shift = 2)

```
Text:    A  B  A  C  A  B  A  B
Pattern:       A  B  A  B
                ↑ starts at index 2
```

Compare:

- P[3] = B vs T[5] = B ✅
    
- P[2] = A vs T[4] = A ✅
    
- P[1] = B vs T[3] = C ❌ mismatch
    

Now:

- matched suffix = `"AB"` (length 2)
    
- so we use **shift[2] = 2**
    

👉 This is where the good suffix rule shines.

Why shift = 2?

- Because `"AB"` appears earlier in the pattern
    
- We align that occurrence
    

✅ **Shift by 2**

---

## ✅ Alignment 4 (shift = 4)

```
Text:    A  B  A  C  A  B  A  B
Pattern:          A  B  A  B
                   ↑ starts at index 4
```

Compare:

- P[3] = B vs T[7] = B ✅
    
- P[2] = A vs T[6] = A ✅
    
- P[1] = B vs T[5] = B ✅
    
- P[0] = A vs T[4] = A ✅
    

🎉 **FULL MATCH**

---

## 🔹 After full match

Now we use:

- **shift[0] = 2**
    

✅ Shift by 2 → continue searching (for overlapping matches)

---

# 🔹 What you should notice

### 1. `shift[4] = 1`

- Used when **no suffix matched**
    
- Just move forward minimally
    

---

### 2. `shift[2] = 2`

- Used when `"AB"` matched
    
- We reused structure inside the pattern
    

---

### 3. `shift[0] = 2`

- Used after full match
    
- Allows overlapping matches (important for patterns like ABAB)
    

---

# 🔹 Where `bpos` fits (intuition)

Your:

- `bpos = [2, 3, 4, 4, 5]`
    

This tells the algorithm:

> “If a suffix doesn’t reappear internally, fall back to the next best border.”

That’s why:

- some shifts are larger (like 4)
    
- some are smaller (like 2)
    

---

# 🔹 The big picture

During matching:

1. You match from right → left
    
2. When mismatch happens:
    
    - figure out how much suffix matched
        
    - jump using `shift[i]`
        
3. `bpos` was used earlier to **build** `shift`
    

---

# 🔹 One-line intuition

👉 **`shift[i]` lets you skip positions by reusing already matched suffix structure instead of restarting blindly.**

---

If you want, next I can:

- derive your exact `shift` array from `bpos` step-by-step
    
- or show how this compares with the **bad character rule** (the other half of Boyer–Moore)