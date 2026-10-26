In Go, it’s worth learning the **`io` / `bufio` / `bytes` / `strings` boundaries** well. A lot of Go I/O code becomes simpler once you choose based on _what kind of data you have_ and _whether you need buffering_.

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

--------------

# io.ReadAll()

In Go, `io.ReadAll()` works the same way for a file and a network connection: it keeps reading until the reader returns `io.EOF` or another error.

The important difference is that a file usually reaches EOF when you reach the end of its contents, whereas a network connection may remain open indefinitely.
## 1. How `io.ReadAll()` works

Consider this code:

go

```
data, err := io.ReadAll(conn)
if err != nil {
    log.Fatal(err)
}
```

Internally, `io.ReadAll()` repeatedly calls `Read()` on the connection. Conceptually, it works like this:

go

```
for {
    n, err := conn.Read(buf)

    data = append(data, buf[:n]...)

    if err != nil {
        if err == io.EOF {
            return data, nil
        }
        return data, err
    }
}
```

This is simplified pseudocode, but it illustrates the behavior.

The `Read()` method returns two important values:

- `n`: the number of bytes read.
    
- `err`: whether an error occurred or the stream has ended.
    

## 2. What happens with a network connection?

In Go, a TCP connection implements the `io.Reader` interface, so you can pass it directly to `io.ReadAll()`.

There are three important scenarios:

|Situation|`Read()` result|What happens?|
|---|---|---|
|Data arrives|`n > 0`, `err == nil`|`io.ReadAll()` appends the data and continues reading.|
|Peer gracefully closes its sending side|Usually `n == 0`, `err == io.EOF`|`io.ReadAll()` stops and returns the accumulated data with a nil error.|
|Network or connection error|A non-nil error other than `io.EOF`|`io.ReadAll()` stops and returns the error.|

One important detail: a `Read()` call can return both `n > 0` and a non-nil error. Go processes the bytes read before handling the error.

## 3. The big gotcha: `io.ReadAll(conn)` can block forever

Suppose you have this server:

go

```
func handleConn(conn net.Conn) {
    defer conn.Close()

    data, err := io.ReadAll(conn)
    if err != nil {
        log.Println("Read error:", err)
        return
    }

    fmt.Println(string(data))
}
```

And a client sends a message:

go

```
conn.Write([]byte("Hello"))
```

What happens?

1. The server reads `"Hello"`.
    
2. The server calls `Read()` again, waiting for more data.
    
3. The client keeps the TCP connection open.
    
4. The server continues waiting because it hasn't received EOF.
    

Writing data does not signal that the message is complete. TCP is a byte stream, not a message-oriented protocol.

The server cannot automatically know whether `"Hello"` is the entire message or just the first part of a larger message.

## 4. How do you know when a network message is complete?

You generally need a protocol to define message boundaries. Common approaches include:

### Approach A: Close the connection to signal completion

go

```
conn.Write([]byte("Hello"))
conn.Close()
```

The receiving side can then read until EOF.

This works when the connection is intended to carry only one stream of data. TCP half-closes are also possible: a client can call `CloseWrite()` on a `*net.TCPConn` to signal that it has finished sending while still receiving data.

### Approach B: Use a delimiter

For example, messages end with a newline:

go

```
reader := bufio.NewReader(conn)

message, err := reader.ReadString('\n')
if err != nil {
    // Handle EOF or another error.
    return
}

fmt.Print(message)
```

This is useful for line-based protocols, provided the delimiter cannot ambiguously occur inside a message.

### Approach C: Prefix the message with its length

For example, send a 4-byte length followed by the message body.

go

```
var header [4]byte

if _, err := io.ReadFull(conn, header[:]); err != nil {
    return
}

length := binary.BigEndian.Uint32(header[:])

data := make([]byte, length)
if _, err := io.ReadFull(conn, data); err != nil {
    return
}

fmt.Println(string(data))
```

Here, `io.ReadFull()` reads exactly the specified number of bytes, unless an error occurs first. The receiver doesn't need to wait for the connection to close.

Remember to validate `length` against a reasonable maximum before allocating memory.

## 5. Mapping C to Go

Your understanding of C is essentially correct. Here's the comparison:

|C `read()`|Go `Read()`|
|---|---|
|Returns a positive number|`n > 0`: bytes were read|
|Returns `0` for EOF|`err == io.EOF`: stream ended|
|Returns `-1` for an error|`err != nil` and `err != io.EOF`: an error occurred|

In Go, check the error rather than checking whether `n == 0` alone:

go

```
n, err := conn.Read(buf)

if n > 0 {
    process(buf[:n])
}

if err != nil {
    if err == io.EOF {
        fmt.Println("Connection's read stream ended")
    } else {
        fmt.Println("Read failed:", err)
    }
}
```

One subtle but important point: `n == 0` with `err == nil` does not necessarily mean EOF. Also, receiving EOF means the peer has finished sending data; it doesn't necessarily mean the entire TCP connection has been closed in both directions.

### The takeaway

- For files, `io.ReadAll()` naturally stops at EOF.
    
- For TCP connections, it stops when the peer signals the end of its sending stream, or a read error occurs.
    
- If the peer keeps the connection open, `io.ReadAll()` keeps waiting.
    
- If you want to read individual messages without closing the connection, define message boundaries using a delimiter, a length prefix, or another protocol rule.

# io.ReadFull()

`io.ReadFull` is a function in Go's standard library (`io` package) that reads exactly the number of bytes you request from an `io.Reader`, unless an error prevents it from doing so.

It's especially useful when reading fixed-size data, such as binary protocol headers, file chunks, or network messages.

## 1. Syntax

go

```
func ReadFull(r Reader, buf []byte) (n int, err error)
```

- `r`: The source to read from, such as a file, network connection, or buffer.
- `buf`: The byte slice to fill.
- `n`: The number of bytes actually read.
- `err`: An error if the requested bytes could not be read completely.

Key idea: Unlike a single call to `Read`, `io.ReadFull` keeps reading until the buffer is full or an error occurs.

## 2. Simple example

go

```
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    reader := strings.NewReader("Hello, World!")
    buf := make([]byte, 5)

    n, err := io.ReadFull(reader, buf)

    fmt.Printf("Read: %q\n", buf[:n])
    fmt.Printf("Bytes: %d\n", n)
    fmt.Printf("Error: %v\n", err)
}
```

Output:

text

```
Read: "Hello"
Bytes: 5
Error: <nil>
```

Even if the underlying reader supplies fewer bytes per individual `Read` call, `io.ReadFull` keeps reading until it fills the 5-byte buffer.

## 3. What happens when there isn't enough data?

Suppose the reader contains only 3 bytes, but you request 5.

go

```
reader := strings.NewReader("ABC")
buf := make([]byte, 5)

n, err := io.ReadFull(reader, buf)

fmt.Println(n)          // 3
fmt.Printf("%q\n", buf) // "ABC\x00\x00"
fmt.Println(err)        // unexpected EOF
```

The buffer has 5 bytes of capacity, but only the first 3 bytes were read. The remaining bytes retain their zero values.

`io.ReadFull` distinguishes between two EOF cases:

Go Packages

| Situation | `n` | Error |
| --- | --- | --- |
| All requested bytes are read | `len(buf)` | `nil` |
| No bytes are available | `0` | `io.EOF` |
| Some bytes are read, but not enough | Partial count | `io.ErrUnexpectedEOF` |

## 4. Difference between `Read` and `ReadFull`

This distinction is particularly important when working with network connections.

go

```
buf := make([]byte, 10)

n, err := conn.Read(buf)
```

A regular `Read` may return just 3 bytes, even if you supplied a 10-byte buffer. That isn't necessarily an error.

By contrast:

go

```
n, err := io.ReadFull(conn, buf)
```

This keeps reading until it gets 10 bytes or encounters an error.

| Feature                                                | `Read`               | `io.ReadFull`                  |
| ------------------------------------------------------ | -------------------- | ------------------------------ |
| Reads the whole buffer when possible                   | Not guaranteed       | Yes                            |
| May return fewer bytes than requested without an error | Yes                  | No, unless the buffer is empty |
| Useful for fixed-size messages                         | Requires extra logic | Yes                            |
### Sample Implementation 

```
func MyReadFull(r io.Reader, buf []byte) (int, error) {
    total := 0

    for total < len(buf) {
        n, err := r.Read(buf[total:])
        total += n

        if err != nil {
            if total == len(buf) {
                return total, nil
            }

            if err == io.EOF && total > 0 {
                return total, io.ErrUnexpectedEOF
            }

            return total, err
        }

        if n == 0 {
            return total, io.ErrNoProgress
        }
    }

    return total, nil
}
```


## 5. Practical example: reading a binary header

Imagine a network protocol where every message starts with a 4-byte header.

go

```
header := make([]byte, 4)

_, err := io.ReadFull(conn, header)
if err != nil {
    return fmt.Errorf("reading header: %w", err)
}

// The complete 4-byte header is now available.
messageLength := binary.BigEndian.Uint32(header)
```

You would also need to import `encoding/binary` and `fmt`.

The important part is that you shouldn't parse the header unless `io.ReadFull` succeeds. Otherwise, you might interpret incomplete data as a valid header.

## 6. When should you use it?

Use `io.ReadFull` when:

- You need an exact number of bytes from a stream.
- You're reading fixed-size binary fields.
- You're implementing a network protocol.
- You need to detect truncated input reliably.

Use `io.ReadAll` instead when you want to read until EOF, and use `io.ReadAtLeast` when you need a minimum number of bytes rather than the entire buffer.

For the official specification, see the [Go `io.ReadFull` documentation](<https://pkg.go.dev/io?utm_source=chatgpt.com#ReadFull>).

Remember: `io.ReadFull` guarantees that the buffer is completely filled only when it returns `err == nil`. It doesn't guarantee that the requested bytes will arrive; the reader may return an error or block while waiting for more data.

