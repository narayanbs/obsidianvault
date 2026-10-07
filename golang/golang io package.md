**`io` is the package that defines the core abstractions for moving bytes around**. Once you understand `io.Reader`, `io.Writer`, and a handful of related interfaces, a huge part of Go's standard library starts making sense.
## 1\. The core idea

 Think of Go I/O as connecting things together:

```
source ──> Reader ──> processing ──> Writer ──> destination
```

 For example:

```
file ──> Reader ──> your program ──> Writer ──> network connection
```

 The important thing is that Go generally doesn't care _what_ the source or destination is. It cares about the interface it provides.

 The two interfaces to know first are:

```
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

 That's basically the foundation of Go I/O.

---
## 2\. `io.Reader`

 A `Reader` produces bytes.

```
func (r Reader) Read(p []byte) (n int, err error)
```

 You give it a byte slice, and it fills that slice with whatever bytes are available.

 Example:

```
data := make([]byte, 5)

n, err := reader.Read(data)

fmt.Println(n)
fmt.Println(data[:n])
```

 If the underlying data is:

```
Hello, world!
```

 the first call might produce:

```
Hello
```

 and the next:

```
, wor
```

 and so on.

 ### Important: `Read` does not mean "read everything"

 This is a common beginner mistake.

 You might write:

```
data := make([]byte, 100)

n, err := r.Read(data)
```

 and expect `n == 100`.

 Not necessarily.

 A `Reader` is allowed to return fewer bytes than the size of `p`.

 So this:

```
n, err := r.Read(buf)
```

 means:

 > "Give me whatever bytes you can put into this buffer right now."

---

## 3\. `io.Writer`

 A `Writer` consumes bytes.

```
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

 For example:

```
data := []byte("hello")

n, err := w.Write(data)
```

 Again, `n` tells you how many bytes were actually written.

 Normally you'll see:

```
n == len(data)
```

 but the interface doesn't guarantee that.

---

## 4\. Why interfaces like this are powerful

 Suppose you write:

```
func copyData(src io.Reader, dst io.Writer) error {
    // ...
}
```

 Your function can now work with:
 ```
- files
- network connections
- strings
- memory buffers
- compressed streams
- HTTP bodies
- encryption streams
- pipes
- custom readers/writers
 ```


 without knowing what any of them actually are.

 For example:

```
func copyData(src io.Reader, dst io.Writer) error {
    _, err := io.Copy(dst, src)
    return err
}
```

 Now:

```
file, _ := os.Open("input.txt")
defer file.Close()

out, _ := os.Create("output.txt")
defer out.Close()

copyData(file, out)
```

 But the exact same function can work with a network connection:

```
io.Copy(conn, file)
```

 or memory:

```
var buf bytes.Buffer

io.Copy(&buf, file)
```

 This is one of the most important design patterns in Go.

---

## 5\. `io.Copy`

 Once you understand `Reader` and `Writer`, this becomes easy:

```
io.Copy(dst, src)
```

 It essentially means:

```
keep reading from src
        ↓
write those bytes to dst
        ↓
repeat until EOF
```

 Conceptually:

```
for {
    n, err := src.Read(buf)

    if n > 0 {
        dst.Write(buf[:n])
    }

    if err == io.EOF {
        break
    }

    if err != nil {
        return err
    }
}
```

 The real implementation has optimizations, but that's the mental model.

---

## 6\. `io.EOF`

 You'll encounter this constantly.

 `EOF` means:

 > There are no more bytes to read.

 For example:

```
for {
    n, err := r.Read(buf)

    if n > 0 {
        process(buf[:n])
    }

    if err == io.EOF {
        break
    }

    if err != nil {
        return err
    }
}
```

 One subtle but important point:

 **`EOF` isn't really a "failure".**

 It's a signal that the stream has ended.

 That's why this is normal:

```
if err == io.EOF {
    // we're done
}
```

 rather than treating it like an exceptional error.

---

## 7\. `io.ReadAll`

 Sometimes you don't want to process a stream incrementally. You just want all of it.

```
data, err := io.ReadAll(r)
```

 For example:

```
data, err := io.ReadAll(resp.Body)
if err != nil {
    return err
}

fmt.Println(string(data))
```

 Conceptually:

```
Reader
  │
  ├── read
  ├── read
  ├── read
  └── EOF
       ↓
 []byte containing everything
```

 ### When not to use it

 Don't blindly do:

```
data, _ := io.ReadAll(hugeFile)
```

 if the input could be gigabytes.

 You're asking your program to hold the entire thing in memory.

 For large data, streaming with `io.Copy` or repeated `Read` calls is usually more appropriate.

---

## 8\. `io.WriteString`

 There's also a useful helper:

```
io.WriteString(w, "hello")
```

 Instead of:

```
w.Write([]byte("hello"))
```

 It's particularly useful because some writers can implement a more efficient string-writing operation.

---

## 9\. `io.LimitReader`

 Suppose you have a reader containing:

```
abcdefghijklmnopqrstuvwxyz
```

 but you only want the first 5 bytes.

```
r := io.LimitReader(src, 5)
```

 Now `r` behaves like a reader that ends after 5 bytes.

 This is particularly useful when dealing with potentially untrusted input.

 For example:

```
r := io.LimitReader(request.Body, 1<<20)
```

 means:

 > Don't let me read more than 1 MiB.

---

## 10\. `io.MultiReader`

 You can combine readers sequentially:

```
r := io.MultiReader(
    strings.NewReader("hello "),
    strings.NewReader("world"),
)
```

 Reading from `r` produces:

```
hello world
```

 It behaves as though the readers were concatenated:

```
Reader A ──> Reader B ──> Reader C
```

---

## 11\. `io.MultiWriter`

 This does the opposite-ish thing.

 You can write to multiple writers simultaneously:

```
w := io.MultiWriter(file, os.Stdout)

fmt.Fprintln(w, "hello")
```

 Now `"hello"` goes to both:

```
              ┌──> file
your Write ───┤
              └──> stdout
```

 This is handy for things like logging to both a file and the console.

---

## 12\. `io.TeeReader`

 This one's especially interesting.

```
r := io.TeeReader(src, dst)
```

 Anything read from `r` is simultaneously written to `dst`.

 Conceptually:

```
             ┌──> your program
src ──> Tee ─┤
             └──> dst
```

 Example:

```
var copy bytes.Buffer

r := io.TeeReader(src, &copy)

data, err := io.ReadAll(r)
```

 Now:
- `data` contains what you read
- `copy` contains a copy of the bytes

 A useful mental model is:

 > "Watch a stream while it passes through."

---

## 13\. `io.Pipe`

 `io.Pipe` lets one goroutine write data while another reads it.

```
r, w := io.Pipe()
```

 You now have:

```
Writer ─────────> Reader
```

 For example:

```
r, w := io.Pipe()

go func() {
    fmt.Fprintln(w, "hello")
    w.Close()
}()

data, _ := io.ReadAll(r)

fmt.Println(string(data))
```

 The interesting part is that the pipe doesn't need to buffer the entire thing.

 The writer and reader can operate concurrently.

 This becomes particularly useful when connecting streaming operations.

---

## 14\. `io.Seeker`

 So far we've talked about streams.

 A `Seeker` lets you move around within something:

```
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}
```

 For example, files implement `io.Seeker`.

```
file.Seek(0, io.SeekStart)
```

 means:

 > Go back to the beginning.

 Other constants include:

```
io.SeekStart
io.SeekCurrent
io.SeekEnd
```

 So you can think of:

```
Reader       → read bytes
Writer       → write bytes
Seeker       → move position
```

 A file can implement all three.

---

## 15\. Interfaces can be combined

 Go frequently combines these small interfaces.

 For example:

```
io.ReadWriter
```

 is essentially:

```
type ReadWriter interface {
    Reader
    Writer
}
```

 Likewise:

```
io.ReadCloser
```

 means:

```
Read + Close
```

 This is why you'll often see things like:

```
func process(r io.ReadCloser)
```

 An HTTP request body, for example, is commonly exposed through an interface combining reading and closing.

---

## 16\. The bigger Go ecosystem

 Once you understand `io`, you'll start seeing the same abstractions everywhere.

 ### Files

```
file, _ := os.Open("foo.txt")
```

 `*os.File` implements `io.Reader`, `io.Writer`, `io.Closer`, `io.Seeker`, etc.

 ### HTTP

```
resp.Body
```

 is an `io.ReadCloser`.

 ### Strings

```
strings.NewReader("hello")
```

 gives you a reader over a string.

 ### Memory

```
var buf bytes.Buffer
```

 `bytes.Buffer` can act as both a reader and writer.

 ### Network

```
conn, _ := net.Dial("tcp", address)
```

 A network connection implements reading and writing.

 So these seemingly unrelated things:

```
file
HTTP body
TCP connection
memory buffer
string
compressed data
```

 can all participate in the same I/O APIs because they implement the same small interfaces.

---

## 17\. The mental model I'd recommend

 Don't try to memorize the entire `io` package.

 Start with this:

```
             produces bytes
                   │
                   ▼
              io.Reader
                   │
                   │
              your program
                   │
                   ▼
              io.Writer
                   │
                   │
              consumes bytes
```

 Then learn these:

| API              | Mental model                  |
| ---------------- | ----------------------------- |
| `io.Reader`      | Give me bytes                 |
| `io.Writer`      | Take these bytes              |
| `io.Copy`        | Stream Reader → Writer        |
| `io.ReadAll`     | Give me the entire stream     |
| `io.EOF`         | Stream is finished            |
| `io.LimitReader` | Read at most N bytes          |
| `io.MultiReader` | Concatenate readers           |
| `io.MultiWriter` | Write to several destinations |
| `io.TeeReader`   | Read while copying elsewhere  |
| `io.Pipe`        | Connect a writer to a reader  |
| `io.Seeker`      | Move around in the stream     |

If you get comfortable with **`Reader → Writer → Copy → EOF`**, you've got the core of Go I/O. The rest of the package becomes much easier to learn.