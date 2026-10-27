
In Go, the `io`, `bufio`, `bytes`, `strings`, and `os` packages are essential for reading, writing, buffering, and manipulating data. Here are the most useful methods, with examples and guidance on when to use them.

# 1. `io` package

Import: `import "io"`

The `io` package provides interfaces and functions for reading and writing data from streams, files, network connections, and other sources.

| Method | Purpose |
| --- | --- |
| `io.ReadAll(r)` | Read all data from a reader |
| `io.Copy(dst, src)` | Copy data from a reader to a writer |
| `io.CopyN(dst, src, n)` | Copy exactly up to `n` bytes |
| `io.CopyBuffer(dst, src, buf)` | Copy using a supplied buffer |
| `io.ReadFull(r, buf)` | Fill a buffer or return an error |
| `io.LimitReader(r, n)` | Limit reading to `n` bytes |
| `io.MultiReader(r1, r2, ...)` | Combine readers sequentially |
| `io.TeeReader(r, w)` | Read data while copying it to a writer |
| `io.MultiWriter(w1, w2, ...)` | Write to multiple writers |
| `io.Discard` | Discard all written data |
| `io.Pipe()` | Connect a reader and writer through a pipe |

### Example: Read all data

go

```
package main

import (
    "fmt"
    "io"
    "strings"
)

func main() {
    r := strings.NewReader("Hello, Go!")

    data, err := io.ReadAll(r)
    if err != nil {
        panic(err)
    }

    fmt.Println(string(data))
}
```

Output:

text

```
Hello, Go!
```

### Example: Copy data

go

```
io.Copy(os.Stdout, strings.NewReader("Hello"))
```

Output:

text

```
Hello
```

### Example: Read a limited number of bytes

go

```
r := strings.NewReader("Hello, World!")

data, err := io.ReadAll(io.LimitReader(r, 5))
if err != nil {
    panic(err)
}

fmt.Println(string(data)) // Hello
```

Important: `io.ReadAll` reads until EOF, so avoid using it without a size limit when reading potentially huge or untrusted input.

# 2. `bufio` package

Import: `import "bufio"`

The `bufio` package adds buffering to readers and writers. It is particularly useful for reading files line by line and reducing the number of underlying I/O operations.

| Method | Purpose |
| --- | --- |
| `bufio.NewReader(r)` | Create a buffered reader |
| `bufio.NewWriter(w)` | Create a buffered writer |
| `bufio.NewScanner(r)` | Create a scanner |
| `Reader.ReadString(delim)` | Read until a delimiter |
| `Reader.ReadBytes(delim)` | Read bytes until a delimiter |
| `Reader.ReadLine()` | Read a line fragment |
| `Reader.ReadByte()` | Read one byte |
| `Reader.Peek(n)` | Inspect upcoming bytes without consuming them |
| `Reader.Discard(n)` | Skip bytes |
| `Writer.WriteString(s)` | Buffer a string |
| `Writer.WriteByte(c)` | Buffer one byte |
| `Writer.Flush()` | Send buffered output to the underlying writer |
| `Scanner.Scan()` | Advance to the next token |
| `Scanner.Text()` | Get the current token as a string |
| `Scanner.Bytes()` | Get the current token as bytes |

### Example: Read a file line by line

go

```
file, err := os.Open("data.txt")
if err != nil {
    panic(err)
}
defer file.Close()

scanner := bufio.NewScanner(file)

for scanner.Scan() {
    fmt.Println(scanner.Text())
}

if err := scanner.Err(); err != nil {
    panic(err)
}
```

By default, `bufio.Scanner` splits input by lines and has a maximum token size of approximately 64 KiB. Increase the limit or use a `bufio.Reader` if you need to process longer lines.

### Example: Read a line using `ReadString`

go

```
reader := bufio.NewReader(
    strings.NewReader("Hello\nWorld\n"),
)

line, err := reader.ReadString('\n')
fmt.Printf("%q %v\n", line, err)
```

Output:

text

```
"Hello\n" <nil>
```

`ReadString` includes the delimiter when it finds one.

### Example: Buffered writing

go

```
writer := bufio.NewWriter(os.Stdout)

writer.WriteString("Hello ")
writer.WriteString("World!")

if err := writer.Flush(); err != nil {
    panic(err)
}
```

Output:

text

```
Hello World!
```

Important: Remember to call `Flush()` when using a buffered writer. Otherwise, some buffered data may never reach the destination.

# 3. `bytes` package

Import: `import "bytes"`

The `bytes` package provides operations for manipulating byte slices (`[]byte`). It is especially useful for binary data, building output efficiently, and working with data read from files or network connections.

| Method | Purpose |
| --- | --- |
| `bytes.NewReader(b)` | Create an `io.Reader` from bytes |
| `bytes.NewBuffer(b)` | Create a mutable byte buffer |
| `bytes.Buffer.Write(p)` | Append bytes |
| `bytes.Buffer.WriteString(s)` | Append a string |
| `bytes.Buffer.String()` | Get buffer contents as a string |
| `bytes.Buffer.Bytes()` | Get buffer contents as a byte slice |
| `bytes.Buffer.Reset()` | Reuse the buffer |
| `bytes.Contains(b, sub)` | Check whether bytes contain a sequence |
| `bytes.Equal(a, b)` | Compare byte slices |
| `bytes.Compare(a, b)` | Compare byte slices lexicographically |
| `bytes.Index(b, sub)` | Find a substring's byte index |
| `bytes.Split(b, sep)` | Split bytes |
| `bytes.ReplaceAll(b, old, new)` | Replace all occurrences |
| `bytes.TrimSpace(b)` | Remove surrounding whitespace |
| `bytes.ToUpper(b)` | Convert ASCII/Unicode-compatible UTF-8 bytes to uppercase |

### Example: Build output using `bytes.Buffer`

go

```
var buf bytes.Buffer

buf.WriteString("Hello")
buf.WriteByte(' ')
buf.WriteString("Go")

fmt.Println(buf.String())
```

Output:

text

```
Hello Go
```

### Example: Search and compare byte slices

go

```
data := []byte("golang programming")

fmt.Println(bytes.Contains(data, []byte("go"))) // true
fmt.Println(bytes.Index(data, []byte("program"))) // 7
fmt.Println(bytes.Equal(data, []byte("golang programming"))) // true
```

### Example: Read bytes through an `io.Reader`

go

```
data := []byte("Hello, Go!")
reader := bytes.NewReader(data)

result, err := io.ReadAll(reader)
if err != nil {
    panic(err)
}

fmt.Println(string(result))
```

When to use `bytes.Buffer`: When you need to construct output incrementally without repeatedly concatenating strings or allocating a new result on every append.

# 4. `strings` package

Import: `import "strings"`

The `strings` package is designed for text processing. Most of its functions work with UTF-8 strings, although indexing and slicing a string operate on bytes rather than Unicode characters.

| Method | Purpose |
| --- | --- |
| `strings.NewReader(s)` | Create an `io.Reader` from a string |
| `strings.Builder` | Efficiently build strings |
| `strings.Contains(s, sub)` | Check whether a substring exists |
| `strings.HasPrefix(s, prefix)` | Check a prefix |
| `strings.HasSuffix(s, suffix)` | Check a suffix |
| `strings.Index(s, sub)` | Find a substring's index |
| `strings.LastIndex(s, sub)` | Find the last occurrence |
| `strings.Split(s, sep)` | Split into a slice |
| `strings.SplitN(s, sep, n)` | Split into at most `n` parts |
| `strings.Fields(s)` | Split on consecutive whitespace |
| `strings.Join(a, sep)` | Join strings |
| `strings.ReplaceAll(s, old, new)` | Replace all occurrences |
| `strings.TrimSpace(s)` | Remove surrounding whitespace |
| `strings.Trim(s, cutset)` | Remove specified characters from both ends |
| `strings.ToUpper(s)` | Convert to uppercase |
| `strings.ToLower(s)` | Convert to lowercase |
| `strings.Count(s, sub)` | Count non-overlapping occurrences |
| `strings.EqualFold(s, t)` | Compare strings case-insensitively |
| `strings.NewReplacer(...)` | Perform multiple replacements |

### Example: Common string operations

go

```
s := "  hello golang world  "

fmt.Println(strings.TrimSpace(s))
fmt.Println(strings.Contains(s, "golang"))
fmt.Println(strings.HasPrefix(s, "  hello"))
fmt.Println(strings.ToUpper(s))
fmt.Println(strings.Fields(s))
```

Output:

text

```
hello golang world
true
true
  HELLO GOLANG WORLD
[hello golang world]
```

### Example: Split and join

go

```
s := "red,green,blue"

parts := strings.Split(s, ",")
fmt.Println(parts)

result := strings.Join(parts, " | ")
fmt.Println(result)
```

Output:

text

```
[red green blue]
red | green | blue
```

### Example: Efficient string building

go

```
var b strings.Builder

b.WriteString("Hello")
b.WriteByte(' ')
b.WriteString("World")

fmt.Println(b.String())
```

Output:

text

```
Hello World
```

`strings.Builder` vs. `bytes.Buffer`:

- Use `strings.Builder` when the final result is text and you want a string.
- Use `bytes.Buffer` when you need byte-oriented operations, a mutable buffer, or an `io.Reader`/`io.Writer` interface.

# 5. `os` package

Import: `import "os"`

The `os` package provides operating-system functionality, including file access, environment variables, directories, and standard input/output.

| Method / value | Purpose |
| --- | --- |
| `os.Open(name)` | Open a file for reading |
| `os.Create(name)` | Create or truncate a file for writing |
| `os.OpenFile(name, flag, perm)` | Open a file with specific flags |
| `os.ReadFile(name)` | Read an entire file |
| `os.WriteFile(name, data, perm)` | Write a file |
| `os.Stat(name)` | Get file information |
| `os.Remove(name)` | Remove a file |
| `os.Rename(old, new)` | Rename a file |
| `os.Mkdir(name, perm)` | Create a directory |
| `os.MkdirAll(path, perm)` | Create directories recursively |
| `os.ReadDir(name)` | List directory entries |
| `os.Getwd()` | Get the current working directory |
| `os.Chdir(dir)` | Change the working directory |
| `os.Getenv(key)` | Get an environment variable |
| `os.Setenv(key, value)` | Set an environment variable |
| `os.LookupEnv(key)` | Get a variable and check whether it exists |
| `os.Exit(code)` | Exit the process immediately |
| `os.Stdin` | Standard input |
| `os.Stdout` | Standard output |
| `os.Stderr` | Standard error |
| `os.Args` | Command-line arguments |

### Example: Read a file

go

```
data, err := os.ReadFile("data.txt")
if err != nil {
    panic(err)
}

fmt.Println(string(data))
```

### Example: Write a file

go

```
data := []byte("Hello from Go!")

err := os.WriteFile("output.txt", data, 0644)
if err != nil {
    panic(err)
}
```

The permission argument `0644` is an octal permission mode on systems that support Unix-style permissions. `os.WriteFile` creates a file or truncates an existing one; it does not append.

### Example: Append to a file

go

```
file, err := os.OpenFile(
    "output.txt",
    os.O_CREATE|os.O_WRONLY|os.O_APPEND,
    0644,
)
if err != nil {
    panic(err)
}
defer file.Close()

if _, err := file.WriteString("New line\n"); err != nil {
    panic(err)
}
```

### Example: Read standard input

go

```
scanner := bufio.NewScanner(os.Stdin)

fmt.Print("Enter your name: ")

if scanner.Scan() {
    fmt.Println("Hello,", scanner.Text())
}

if err := scanner.Err(); err != nil {
    panic(err)
}
```

### Example: Read environment variables

go

```
value, exists := os.LookupEnv("HOME")

if exists {
    fmt.Println("Home:", value)
} else {
    fmt.Println("HOME is not set")
}
```

# 6\. How these packages work together

A useful way to understand Go I/O is to think of the packages as different layers.

### `os`

Files, directories, stdin, stdout

### `io`

Reader and Writer interfaces; copying streams

### `bufio`

Buffering, line-by-line reading, efficient writing

### `bytes`

Byte slices, binary data, byte buffers

### `strings`

Text processing, string readers, string builders

These packages overlap in useful ways. For example, an `*os.File`, a `*bytes.Buffer`, and a `*strings.Reader` can all serve as readers or writers where their respective interfaces are implemented.

## Example: Copy a file efficiently

This example combines `os`, `bufio`, and `io`.

go

```
package main

import (
    "bufio"
    "io"
    "os"
)

func main() {
    src, err := os.Open("input.txt")
    if err != nil {
        panic(err)
    }
    defer src.Close()

    dst, err := os.Create("output.txt")
    if err != nil {
        panic(err)
    }
    defer dst.Close()

    writer := bufio.NewWriter(dst)
    if _, err := io.Copy(writer, src); err != nil {
        panic(err)
    }

    if err := writer.Flush(); err != nil {
        panic(err)
    }
}
```

`io.Copy` handles the stream transfer, while `bufio.NewWriter` buffers output. For ordinary file-to-file copying, `io.Copy(dst, src)` alone is often sufficient because Go can use optimized copy paths.

# 7\. Quick reference: Which method should you use?

| Task | Recommended method |
| --- | --- |
| Read an entire small file | `os.ReadFile()` |
| Write an entire file | `os.WriteFile()` |
| Read a large file as a stream | `os.Open()` \+ `io.Copy()` or buffered reads |
| Read a file line by line | `bufio.Scanner` |
| Read a potentially long line | `bufio.Reader.ReadString()` |
| Read only a limited amount | `io.LimitReader()` |
| Copy data between streams | `io.Copy()` |
| Build a string incrementally | `strings.Builder` |
| Build or manipulate bytes | `bytes.Buffer` |
| Search text | `strings.Contains()` / `strings.Index()` |
| Search binary data | `bytes.Contains()` / `bytes.Index()` |
| Split text by whitespace | `strings.Fields()` |
| Split byte data | `bytes.Split()` |
| Write to stdout | `fmt.Print*()` or `io.Copy(os.Stdout, r)` |
| Read user input | `bufio.Scanner` with `os.Stdin` |
| Append to a file | `os.OpenFile()` with `os.O_APPEND` |
| Read multiple sources sequentially | `io.MultiReader()` |
| Write to multiple destinations | `io.MultiWriter()` |

## 8\. Five important concepts to remember

1. `io.Reader` reads data into a byte slice; `io.Writer` writes bytes from a byte slice.
2. `bufio` improves convenience and can reduce I/O operations through buffering.
3. `bytes` is for byte-oriented operations; `strings` is for text-oriented operations.
4. `os` connects your program to files, directories, environment variables, and standard streams.
5. Always handle errors, close opened files with `defer file.Close()`, and check `scanner.Err()` after scanning.

If you're learning Go for backend development, start by mastering `io.Reader`, `io.Writer`, `io.Copy`, `bufio.Scanner`, `os.Open`, `os.ReadFile`, `bytes.Buffer`, and `strings.Builder`. These provide a strong foundation for file processing, HTTP handlers, logging, and stream-based programming.