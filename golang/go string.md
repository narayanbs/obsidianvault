## 1. What happens with `s := "hello"`?

go

```go
s := "hello"
```

Go stores the string as UTF-8 encoded bytes.

The characters have these Unicode code points:

| Character | Unicode code point | UTF-8 bytes |
| --- | --- | --- |
| h | U+0068 | 68 |
| e | U+0065 | 65 |
| l | U+006C | 6C |
| l | U+006C | 6C |
| o | U+006F | 6F |

So the underlying bytes are:

go

```go
[0x68, 0x65, 0x6c, 0x6c, 0x6f]
```

Each character happens to take exactly one byte in this example.

## 2. What happens with non-ASCII characters?

Consider:

go

```go
s := "hé"
```

Here:

- `h` has code point U+0068 and takes 1 byte.
- `é` has code point U+00E9 and takes 2 bytes in UTF-8.

The bytes are:

go

```go
[0x68, 0xC3, 0xA9]
```

Notice something important:

The number of bytes is not necessarily equal to the number of characters (Unicode code points).

## 3. How does Go expose this?

Go gives you two different ways to iterate over a string.

A. Iterate over bytes

go

```go
s := "hé"

for i := 0; i < len(s); i++ {
    fmt.Printf("%x ", s[i])
}
```

Output:

text

```text
68 c3 a9
```

Here you're iterating over individual bytes.

B. Iterate over Unicode code points (runes)

go

```go
s := "hé"

for i, r := range s {
    fmt.Printf("index=%d, rune=%U\n", i, r)
}
```

Output:

text

```text
index=0, rune=U+0068
index=1, rune=U+00E9
```

Here:

- `r` is a `rune`, which is an alias for `int32`.
- `range` decodes the UTF-8 bytes into Unicode code points.
- `i` is the byte index, not the character index.

## 4. One more important distinction: Unicode code points vs characters

A Unicode code point is not always equivalent to what humans perceive as a character.

For example:

go

```go
s := "é"
```

This can be represented in two ways:

- U+00E9 (one code point)
- U+0065 followed by U+0301 (`e` + combining acute accent, two code points)

Both can look identical on screen.

So there are actually three concepts:

| Concept                     | Meaning                                                              |
| --------------------------- | -------------------------------------------------------------------- |
| Byte                        | Individual 8-bit unit stored in a Go string                          |
| Unicode code point (`rune`) | A Unicode value representing a code point                            |
| Grapheme cluster            | A user-perceived character, potentially made of multiple code points |

## 5. The mental model to remember
```text
Go string ("hé")
      |
      v
UTF-8 encoded bytes
[0x68, 0xC3, 0xA9]
      |
      v
UTF-8 decoding
      |
      v
Unicode code points (runes)
[U+0068, U+00E9]
      |
      v
Grapheme clusters
["h", "é"]

```


The key takeaway: Your mental model is correct. Think of a Go string as UTF-8 encoded bytes. When you need Unicode code points, decode those bytes into `rune`s. When you need user-perceived characters, you need grapheme-cluster processing.

One particularly useful Go rule: `len(s)` returns the number of bytes, whereas `utf8.RuneCountInString(s)` returns the number of Unicode code points.