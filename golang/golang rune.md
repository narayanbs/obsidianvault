### rune represents a unicode code point, but are the bytes in the rune encoded in UTF-8


You're very close, but there's one important correction:

> A **rune does not contain UTF-8 bytes**.

A `rune` is just an integer value (`int32`) representing a Unicode code point. UTF-8 is only one way of **encoding** that code point into bytes.

For example:

```go
r := 'é'
```

The value of `r` is:

```text
U+00E9
```

Internally, `r` is stored as an `int32`:

```text
decimal: 233
hex:     0x00E9
```

There are **no UTF-8 bytes inside the rune**.

If you want to store or transmit that rune as text, Go encodes it into UTF-8:

```go
s := string(r)
fmt.Printf("% x\n", []byte(s))
```

Output:

```text
c3 a9
```

Those are the UTF-8 bytes for the code point `U+00E9`.

---

### Another example

```go
r := '😀'
```

The rune's value is

```text
U+1F600
```

As an `int32`, that's just the integer:

```text
0x1F600
```

When converted to a string:

```go
s := string(r)
```

the string contains these UTF-8 bytes:

```text
f0 9f 98 80
```

---

### A helpful mental model

Think of it like this:

```
        Unicode
      code point
          │
          ▼
      rune (int32)
          │
          │ encode as UTF-8
          ▼
      bytes: F0 9F 98 80
```

The rune is the **abstract Unicode code point represented as an integer**. UTF-8 is an **encoding** that turns that integer into a sequence of bytes.

### In Go

```go
r := 'A'        // rune (int32)
s := string(r)  // UTF-8 encoded string
b := []byte(s)  // UTF-8 bytes
```

So a more accurate statement is:

> **A rune is a Unicode code point stored as an `int32`. When a rune is placed into a string, Go encodes it as UTF-8.**

That's the distinction that makes Go's handling of text much easier to reason about.

------------------------
###  A rune literal is expressed as one or more characters enclosed in a pair of quotes. The enclosed characters denote one Unicode code point value.

### Question : so a rune literal is equal to one unicode code point.  But i have read that a character can span multiple runes.  In that case how do we specify that as a rune literal

This is a really common point of confusion because Go documentation uses the word **character** in a slightly different way than people often do in everyday language.

The key is to distinguish three different concepts:

1. **Unicode code point** (what a Go `rune` represents)
    
2. **Grapheme cluster** (what a user perceives as a single character)
    
3. **UTF-8 encoding** (how code points are stored in bytes)
    

### A rune is exactly one Unicode code point

In Go,

```go
var r rune = 'A'
```

or

```go
r := '😀'
```

Both of these are a **single rune**, because each corresponds to one Unicode code point.

```go
fmt.Printf("%U\n", 'A')
// U+0041

fmt.Printf("%U\n", '😀')
// U+1F600
```

Even though 😀 takes 4 bytes in UTF-8, it is still **one rune**.

---

### But what about a "character" that spans multiple runes?

Here "character" usually means a **grapheme cluster**—what a human sees as one symbol.

For example:

```
é
```

can be represented in two different ways.

#### 1. Single code point

```
U+00E9
```

Go literal:

```go
r := 'é'
```

One rune.

---

#### 2. Two code points

```
U+0065   LATIN SMALL LETTER E
U+0301   COMBINING ACUTE ACCENT
```

Visually it still looks like

```
é
```

but it is **two runes**.

You cannot write this as a rune literal because it isn't one code point.

Instead you use a string:

```go
s := "é"
```

Here:

```go
fmt.Println(len([]rune(s)))
```

prints

```
2
```

---

### Another example

The family emoji

```
👨‍👩‍👧‍👦
```

looks like one "character", but it's actually several code points:

```
👨
ZWJ
👩
ZWJ
👧
ZWJ
👦
```

That's multiple runes.

So you write

```go
s := "👨‍👩‍👧‍👦"
```

not

```go
r := '👨‍👩‍👧‍👦' // invalid
```

because a rune literal must contain exactly one Unicode code point.

---

### Why does the Go spec say "one or more characters"?

The wording refers to the **source text inside the quotes**, not the number of Unicode code points represented.

For example, these are all valid rune literals:

```go
'A'
'\n'
'\x41'
'\u00E9'
'\U0001F600'
```

Notice that:

- `'A'` contains one source character.
    
- `'\U0001F600'` contains **ten source characters** (`\`, `U`, `0`, `0`, `0`, `1`, `F`, `6`, `0`, `0`), but they collectively denote **one Unicode code point**.
    

So "one or more characters enclosed in quotes" means the literal's _text_ may be multiple source characters because of escape sequences, but after those escapes are interpreted, the literal must represent exactly **one Unicode code point**.

### Rule of thumb

- **`rune`** = exactly **one Unicode code point**.
    
- **Rune literal (`'...'`)** = must denote exactly **one Unicode code point**.
    
- **String (`"..."`)** = zero or more Unicode code points, so it can represent grapheme clusters that consist of multiple runes.
    

So if something that looks like one visible character is actually composed of multiple Unicode code points, there is **no such thing as a rune literal for it**. You represent it as a string, and if needed, iterate over its constituent runes.


### strings and runes

In Go, a `string` is essentially a sequence of **bytes**. By convention, those bytes usually contain **UTF-8 encoded text**.

A `rune` represents a **Unicode code point**.

## The basic idea

```text
string
   ↓ UTF-8 decoding
[]rune
   ↓
Unicode code points

string

A Go string is a sequence of bytes.

For example:

s := "hello你好"


The string contains:

h e l l o 你 好


But internally, it is stored as UTF-8 bytes.

In UTF-8:

h   → 1 byte
e   → 1 byte
l   → 1 byte
l   → 1 byte
o   → 1 byte
你  → 3 bytes
好  → 3 bytes


Therefore:

len(s)


returns the number of bytes, not the number of characters.

fmt.Println(len(s))


Output:

11


Because:

5 ASCII characters × 1 byte = 5 bytes
2 Chinese characters × 3 bytes = 6 bytes

Total = 11 bytes

rune

In Go:

type rune = int32


A rune represents a Unicode code point.

For example:

你 → U+4F60
好 → U+597D


The numeric values are:

你 → 20320
好 → 22909

Converting a string to []rune

When we do:

runes := []rune(s)


Go decodes the UTF-8 string and gives us a slice of Unicode code points.

For:

s := "hello你好"


we get:

[]rune(s)


which is approximately:

[104 101 108 108 111 20320 22909]


There are 7 runes:

h  → 104
e  → 101
l  → 108
l  → 108
o  → 111
你 → 20320
好 → 22909


Therefore:

len(s)           // 11 bytes
len([]rune(s))   // 7 runes

Why doesn't fmt.Println([]rune(s)) print the characters?

If you do:

fmt.Println([]rune(s))


you'll get:

[104 101 108 108 111 20320 22909]


That's because you're printing a slice of runes, and rune is an integer type.

If you want to print each rune as a character:

for _, r := range []rune(s) {
    fmt.Printf("%c ", r)
}


Output:

h e l l o 你 好


Alternatively:

fmt.Println(string([]rune(s)))


Output:

hello你好

What actually happens with 你?

The character:

你


is represented in UTF-8 using 3 bytes:

E4 BD A0


When Go decodes those bytes into a rune:

E4 BD A0
    ↓
UTF-8 decoding
    ↓
rune: 20320
    ↓
Unicode code point: U+4F60
    ↓
displayed character: 你


So we can think of it as:

UTF-8 bytes
    ↓
Unicode code point
    ↓
character displayed on screen

Important distinction: bytes vs runes
Go expression	What it represents
string	Sequence of bytes
len(s)	Number of bytes
[]byte(s)	Individual UTF-8 bytes
rune	Unicode code point
[]rune(s)	Slice of Unicode code points
len([]rune(s))	Number of Unicode code points
string([]rune(s))	UTF-8 string reconstructed from those code points
One subtle point: rune ≠ "human character"

A rune represents a Unicode code point, but a Unicode code point isn't always the same thing as what a human perceives as one character.

For example, a visible character can sometimes be made from multiple Unicode code points:

letter + combining accent


Some emoji can also consist of multiple code points:

👨‍👩‍👧‍👦


So:

len([]rune(s))


counts Unicode code points, not necessarily the number of user-perceived characters.

The latter are called grapheme clusters.

Mental model

The easiest way to remember it is:

                 UTF-8
string  ──────────────────────→ bytes
  │
  │ UTF-8 decoding
  ↓
[]rune ───────────────────────→ Unicode code points
  │
  │ rendered by a font
  ↓
characters you see on screen


So:

len(string) → number of bytes

[]rune(string) → Unicode code points

len([]rune(string)) → number of Unicode code points

string([]rune(s)) → convert the code points back into a UTF-8 string


**Important:** because the whole thing is inside one Markdown code block, the inner triple-backtick code blocks will prevent it from rendering if you paste it exactly as-is.

For the cleanest Obsidian result, **copy the content from `# Go Strings...` onward, but not the outer ```markdown and final ```**.