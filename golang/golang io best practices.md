In Go, it’s worth learning the **`io` / `bufio` / `bytes` / `strings` boundaries** well. A lot of Go I/O code becomes simpler once you choose based on _what kind of data you have_ and _whether you need buffering_.

 ## The mental model

 Think of the packages roughly like this:

| Package   | Think of it as                                 | Typical use                                        |
| --------- | ---------------------------------------------- | -------------------------------------------------- |
| `io`      | **I/O plumbing & composition**                 | Copying, limiting, reading exact amounts, adapters |
| `bufio`   | **Buffered I/O + line/token-oriented helpers** | Files, sockets, stdin, repeated small reads/writes |
| `bytes`   | **In-memory `[]byte` operations**              | Buffers, parsing binary data, building byte output |
| `strings` | **In-memory `string` operations**              | Text parsing/manipulation                          |
| `os`      | **Operating-system resources**                 | Files, stdin/stdout, pipes                         |
| `fmt`     | **Human-oriented formatting/parsing**          | CLI output, debugging, simple formatted I/O        |

The most important distinction is:

 > **`bufio` is not "better I/O". It is buffered I/O. Don't add it automatically.**

---

# 1\. Use `io` utilities more often than you probably think

 The `io` package is deliberately small and composable.

 For example, instead of manually doing:

```
buf := make([]byte, 32*1024)

for {
    n, err := r.Read(buf)
    if n > 0 {
        // process buf[:n]
    }

    if err == io.EOF {
        break
    }
    if err != nil {
        return err
    }
}
```

 you can often use:

```
_, err := io.Copy(dst, src)
```

 This is idiomatic Go.

 ### Some `io` functions worth knowing extremely well

```
io.Copy
io.CopyN
io.LimitReader
io.ReadAll
io.ReadFull
io.WriteString
io.Discard
io.MultiReader
io.MultiWriter
```

 And interfaces:

```
io.Reader
io.Writer
io.Closer
io.Seeker
io.ReaderAt
io.WriterAt
io.ReadWriter
```

 These interfaces are the foundation of Go's I/O design.

---

 # 2\. `io.Copy` should be your default for copying streams

 Instead of:

```
data, err := io.ReadAll(src)
if err != nil {
    return err
}

_, err = dst.Write(data)
return err
```

 prefer:

```
_, err := io.Copy(dst, src)
return err
```

 Why?

 Your version requires:

```
source → memory → destination
```

 while `io.Copy` can do:

```
source → destination
```

 without requiring the entire stream in memory.

 It also gives implementations an opportunity to use optimized interfaces such as `WriterTo` and `ReaderFrom`.

 For example:

```
func copyFile(src, dst string) error {
    in, err := os.Open(src)
    if err != nil {
        return err
    }
    defer in.Close()

    out, err := os.Create(dst)
    if err != nil {
        return err
    }
    defer out.Close()

    _, err = io.Copy(out, in)
    return err
}
```

 This is much more idiomatic than manually allocating a buffer.

---

 # 3\. When should you use `bufio`?

 Use `bufio` when **buffering actually helps**, or when you want its higher-level APIs.

 There are two major reasons.

 ## A. You have many small I/O operations

 For example:

```
f, err := os.Create("output.txt")
if err != nil {
    return err
}
defer f.Close()

w := bufio.NewWriter(f)
defer w.Flush()

for i := 0; i < 100000; i++ {
    fmt.Fprintln(w, i)
}
```

 Without buffering, you may perform a huge number of small writes.

 With:

```
bufio.NewWriter(f)
```

 those writes are accumulated and flushed in larger chunks.

---

 # 4\. `bufio.Reader` is useful for structured stream reading

 A classic example:

```
r := bufio.NewReader(file)

line, err := r.ReadString('\n')
```

 or:

```
line, err := r.ReadBytes('\n')
```

 Even better for many ordinary line-oriented cases:

```
scanner := bufio.NewScanner(file)

for scanner.Scan() {
    line := scanner.Text()
    // process line
}

if err := scanner.Err(); err != nil {
    return err
}
```

 So:

```
Need lines/tokens?
        ↓
    bufio.Scanner

Need buffered arbitrary reads?
        ↓
    bufio.Reader

Need buffered writes?
        ↓
    bufio.Writer
```

---

 # 5\. Don't blindly wrap everything in `bufio`

 This is a common beginner pattern:

```
reader := bufio.NewReader(file)

data, err := io.ReadAll(reader)
```

 It's often unnecessary.

 You can simply do:

```
data, err := io.ReadAll(file)
```

 Likewise:

```
w := bufio.NewWriter(file)
io.Copy(w, src)
w.Flush()
```

 may be unnecessary depending on what you're doing.

 A good rule:

 > **Use `bufio` because you need buffering or its higher-level API, not because "I/O should always be buffered."**

---

 # 6\. `bufio.Scanner` has an important limitation

 This catches people surprisingly often.

 `Scanner` has a maximum token size by default.

 So this can fail for very large lines:

```
scanner := bufio.NewScanner(r)

for scanner.Scan() {
    ...
}
```

 If you expect large lines:

```
scanner := bufio.NewScanner(r)
scanner.Buffer(make([]byte, 64*1024), 10*1024*1024)
```

 Or use `bufio.Reader` when you need more control.

 For example:

```
r := bufio.NewReader(file)

for {
    line, err := r.ReadBytes('\n')
    if len(line) > 0 {
        process(line)
    }

    if err == io.EOF {
        break
    }
    if err != nil {
        return err
    }
}
```

---

 # 7\. `bytes` is for data that is already in memory

 This distinction is extremely useful:

```
stream             → io
buffered stream    → bufio
[]byte in memory   → bytes
string in memory   → strings
```

 For example:

```
data := []byte("hello, world")

if bytes.HasPrefix(data, []byte("hello")) {
    ...
}
```

 Don't convert unnecessarily:

```
if strings.HasPrefix(string(data), "hello") {
    ...
}
```

 Use:

```
bytes.HasPrefix(data, []byte("hello"))
```

 if you already have `[]byte`.

---

 # 8\. `bytes.Buffer` is particularly important

 `bytes.Buffer` is an in-memory buffer implementing `io.Reader` and `io.Writer`.

 For example:

```
var buf bytes.Buffer

buf.WriteString("hello ")
buf.WriteString("world")

fmt.Println(buf.String())
```

 It's useful when you're **building data in memory incrementally**.

 For example:

```
var buf bytes.Buffer

for _, item := range items {
    buf.WriteString(item)
    buf.WriteByte('\n')
}

return buf.Bytes()
```

 This is generally preferable to repeatedly doing:

```
result += item
```

 for large amounts of data.

---

 # 9\. `bytes.Buffer` vs `strings.Builder`

 This is another useful distinction.

 If you're producing **text**:

```
var b strings.Builder

b.WriteString("hello")
b.WriteString(" world")

return b.String()
```

 If you're producing **arbitrary bytes**:

```
var b bytes.Buffer

b.Write([]byte{0x01, 0x02, 0x03})
```

 So roughly:

```
Building string
    ↓
strings.Builder

Building []byte / byte-oriented data
    ↓
bytes.Buffer
```

 Don't use `bytes.Buffer` simply because you've seen it everywhere.

---

 # 10\. `bytes.Reader` is the opposite direction

 `bytes.Buffer` is mainly useful for **writing/building** data.

 `bytes.Reader` is useful for **reading existing `[]byte` data through an `io.Reader` interface**.

 For example:

```
data := []byte("hello world")

r := bytes.NewReader(data)

_, err := io.Copy(dst, r)
```

 This is particularly useful when an API expects an `io.Reader` but your data is already in memory.

---

 # 11\. `strings.NewReader` is the same idea for strings

 Instead of:

```
data := []byte("hello world")
r := bytes.NewReader(data)
```

 you can do:

```
r := strings.NewReader("hello world")
```

 This is a very nice Go pattern:

```
func process(r io.Reader) error {
    ...
}

process(strings.NewReader("hello world"))
```

 You don't need to manufacture a file or byte slice just to satisfy `io.Reader`.

---

 # 12\. `io.ReadAll` is great — when the data is reasonably bounded

 This is perfectly idiomatic:

```
data, err := io.ReadAll(r)
if err != nil {
    return err
}
```

 But remember what it means:

 > "Give me the entire stream in memory."

 That's appropriate for things like:

 - small HTTP responses
- configuration files
- small JSON documents
- test fixtures
- small request bodies

 Less appropriate for:

 - uploaded files
- huge files
- untrusted request bodies
- potentially infinite streams

 For untrusted/bounded input, combine it with:

```
data, err := io.ReadAll(io.LimitReader(r, 10<<20))
```

 That means:

 > Read at most 10 MiB.

 This combination is extremely useful.

---

 # 13\. `io.LimitReader` is underrated

 Suppose an HTTP endpoint expects a JSON document no larger than 1 MB.

 Instead of:

```
body, err := io.ReadAll(r.Body)
```

 you can establish a limit:

```
body, err := io.ReadAll(io.LimitReader(r.Body, 1<<20))
```

 Or, for HTTP servers, `http.MaxBytesReader` is often even more appropriate for request bodies.

 The important conceptual pattern is:

```
Reader
  ↓
LimitReader
  ↓
your parser
```

 That's the beauty of Go's composable interfaces.

---

 # 14\. `io.MultiReader` and `io.MultiWriter`

 These are surprisingly useful.

 ### Concatenate readers

```
r := io.MultiReader(
    strings.NewReader("hello "),
    strings.NewReader("world"),
)
```

 From the consumer's perspective:

```
hello world
```

 It's effectively:

```
reader A → reader B → reader C
```

 without manually copying everything into a new buffer.

 ### Write to multiple destinations

```
w := io.MultiWriter(file, logFile)

fmt.Fprintln(w, "important event")
```

 Now the write goes to both.

---

 # 15\. `io.TeeReader` is useful for "read while doing something else"

 For example, suppose you want to calculate a hash while copying:

```
h := sha256.New()

reader := io.TeeReader(src, h)

_, err := io.Copy(dst, reader)
if err != nil {
    return err
}

sum := h.Sum(nil)
```

 Conceptually:

```
             ┌──→ hash
source ──────┤
             └──→ destination
```

 without separately reading the source twice.

 This is one of the nicest examples of Go's `io.Reader` composition.

---

 # 16\. Don't use `fmt` for high-volume I/O by default

 This:

```
fmt.Fprintf(w, "user=%s id=%d\n", user, id)
```

 is perfectly fine for many applications.

 But don't assume `fmt` is the optimal way to serialize large amounts of data.

 For performance-sensitive code, consider:

```
w.Write(...)
```

 or:

```
strconv.AppendInt(...)
strconv.AppendFloat(...)
```

 or specialized encoders.

 The important principle is:

 > **Optimize after profiling.**

 Don't replace readable `fmt.Fprintf` everywhere just because it might be slower.

---

 # 17\. `bytes` vs `strings`: don't convert back and forth

 A common smell:

```
data []byte

s := string(data)
s = strings.TrimSpace(s)
data = []byte(s)
```

 Sometimes that's completely fine.

 But if you're fundamentally processing bytes, consider:

```
data = bytes.TrimSpace(data)
```

 Similarly, if your domain is fundamentally text:

```
s := strings.TrimSpace(s)
```

 rather than converting to bytes.

 Conversions aren't automatically terrible, but unnecessary conversions add allocations/copies and obscure intent.

---

 # 18\. A useful decision tree

 When you're writing I/O code, ask these questions:

 ### "Do I have a stream?"

 Use:

```
io.Reader
io.Writer
```

 and the `io` package.

---

 ### "Do I need to read/write many small chunks?"

 Consider:

```
bufio.Reader
bufio.Writer
```

---

 ### "Do I need lines?"

 Usually:

```
bufio.Scanner
```

 or:

```
bufio.Reader
```

 depending on size/control requirements.

---

 ### "Do I already have all the data as `[]byte`?"

 Use:

```
bytes.*
```

---

 ### "Do I already have all the data as `string`?"

 Use:

```
strings.*
```

---

 ### "Do I need to construct a large string?"

 Use:

```
strings.Builder
```

---

 ### "Do I need to construct binary/byte data?"

 Use:

```
bytes.Buffer
```

 or, depending on the problem, append directly to a `[]byte`.

---

 ### "Do I simply want to transfer data?"

 Start with:

```
io.Copy(dst, src)
```

---

 ### "Do I need all the data?"

 Use:

```
io.ReadAll(r)
```

 **only if the size is appropriate/controlled.**

---

 # 19\. One particularly Go-ish pattern

 Suppose you have:

```
func process(r io.Reader) error
```

 That's usually much more reusable than:

```
func process(filename string) error
```

 Because now callers can give you:

```
process(file)
```

 or:

```
process(bytes.NewReader(data))
```

 or:

```
process(strings.NewReader(text))
```

 or:

```
process(httpResponse.Body)
```

 or even:

```
process(gzipReader)
```

 Your function doesn't care.

 This is one of the central ideas behind Go's I/O ecosystem:

 > **Program against `io.Reader`/`io.Writer`, not concrete sources and destinations.**

---

 # 20\. My practical rules of thumb

 If I were making a Go codebase guideline, I'd use something close to this:

```
                ┌─────────────────┐
                │   What do I have?│
                └────────┬────────┘
                         │
             ┌───────────┴───────────┐
             │                       │
          stream                  memory
             │                       │
        ┌────┴────┐            ┌─────┴─────┐
        │         │            │           │
      read      write       []byte       string
        │         │            │           │
     bufio?    bufio?       bytes       strings
        │         │
        └────┬────┘
             │
          io.*
```

 And specifically:

 - **Prefer `io.Copy` over manually copying streams.**
- **Use `io.Reader`/`io.Writer` in APIs whenever appropriate.**
- **Don't wrap everything in `bufio`.**
- **Use `bufio.Scanner` for convenient line/token scanning, but remember its token-size limit.**
- **Use `bufio.Reader/Writer` when buffering or their APIs provide value.**
- **Use `io.ReadAll` when the entire input is intentionally being loaded into memory.**
- **Use `io.LimitReader` when input size needs a hard bound.**
- **Use `bytes` for byte-oriented in-memory work.**
- **Use `strings` for string-oriented in-memory work.**
- **Use `bytes.Buffer` / `strings.Builder` when incrementally constructing data.**
- **Avoid needless `string ↔ []byte` conversions.**
- **Don't optimize I/O abstractions prematurely; profile first.**

 The deeper lesson is that **`bufio` isn't the center of Go I/O — `io.Reader` and `io.Writer` are**. `bufio`, `bytes`, `strings`, files, sockets, compression readers, HTTP bodies, etc. all plug into that same abstraction. Once you start thinking in terms of _composable readers/writers_, Go I/O becomes much easier to design.