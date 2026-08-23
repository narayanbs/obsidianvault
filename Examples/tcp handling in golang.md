The key idea is: **don't assume one `Read()` equals one message**. If you know you need to read, say, **N bytes**, keep reading until you've received all N bytes.

### 1. If you know exactly how many bytes you need

Go already provides exactly what you want: `io.ReadFull`.

```go
func readN(conn net.Conn, n int) ([]byte, error) {
    buf := make([]byte, n)

    _, err := io.ReadFull(conn, buf)
    if err != nil {
        return nil, err
    }

    return buf, nil
}
```

Usage:

```go
data, err := readN(conn, 10)
if err != nil {
    return err
}

fmt.Printf("received: %q\n", data)
```

Even if the sender sends those 10 bytes in **5 separate TCP packets**, `io.ReadFull` keeps reading until all 10 bytes have arrived.

---

## 2. Better: length-prefixed protocol

This is probably what you want if you're designing your own protocol.

For example, define your protocol as:

```text
+----------------+----------------------+
| 4-byte length  |     payload          |
+----------------+----------------------+
```

So if you want to send:

```text
"hello world"
```

you send:

```text
00 00 00 0B    h e l l o   w o r l d
^^^^^^^^^^^^   ^^^^^^^^^^^^^^^^^^^^^
length = 11       11 bytes
```

Then your reader can be:

```go
func readMessage(conn net.Conn) ([]byte, error) {
    // First read the 4-byte length.
    var lengthBuf [4]byte

    if _, err := io.ReadFull(conn, lengthBuf[:]); err != nil {
        return nil, err
    }

    length := binary.BigEndian.Uint32(lengthBuf[:])

    // Optional: protect yourself from malicious/invalid lengths.
    if length > 10*1024*1024 {
        return nil, fmt.Errorf("message too large: %d bytes", length)
    }

    // Now read exactly `length` bytes.
    message := make([]byte, length)

    if _, err := io.ReadFull(conn, message); err != nil {
        return nil, err
    }

    return message, nil
}
```

You'd use it like:

```go
for {
    message, err := readMessage(conn)
    if err != nil {
        if errors.Is(err, io.EOF) {
            fmt.Println("client disconnected")
        } else {
            fmt.Println("read error:", err)
        }
        return
    }

    fmt.Printf("received: %q\n", message)
}
```

This is a very common pattern.

---

## 3. If your protocol is text-based

If your messages are terminated by `\n`, then `bufio.Reader` is convenient:

```go
reader := bufio.NewReader(conn)

for {
    line, err := reader.ReadString('\n')
    if err != nil {
        if errors.Is(err, io.EOF) {
            fmt.Println("client disconnected")
        }
        return
    }

    fmt.Printf("received: %q\n", line)
}
```

For example, the client sends:

```text
hello\n
world\n
this is another message\n
```

and your loop receives each message separately.

You can also use:

```go
line, err := reader.ReadBytes('\n')
```

or:

```go
line, err := reader.ReadString('\n')
```

depending on whether you want `[]byte` or `string`.

---

## 4. If you have multiple fields in a message

Suppose your protocol is:

```text
[4 bytes: user ID]
[2 bytes: message length]
[N bytes: message]
```

Then I'd structure the reader like this:

```go
func readMessage(conn net.Conn) (uint32, string, error) {
    var userIDBuf [4]byte

    if _, err := io.ReadFull(conn, userIDBuf[:]); err != nil {
        return 0, "", err
    }

    userID := binary.BigEndian.Uint32(userIDBuf[:])

    var lengthBuf [2]byte

    if _, err := io.ReadFull(conn, lengthBuf[:]); err != nil {
        return 0, "", err
    }

    length := binary.BigEndian.Uint16(lengthBuf[:])

    messageBuf := make([]byte, length)

    if _, err := io.ReadFull(conn, messageBuf); err != nil {
        return 0, "", err
    }

    return userID, string(messageBuf), nil
}
```

Now your connection handler stays very clean:

```go
func handleConn(conn net.Conn) {
    defer conn.Close()

    for {
        userID, message, err := readMessage(conn)
        if err != nil {
            if errors.Is(err, io.EOF) {
                return
            }

            fmt.Println("read error:", err)
            return
        }

        fmt.Println("user:", userID)
        fmt.Println("message:", message)
    }
}
```

### The important mental model

Think of TCP like this:

```text
Your application:
    Message A
    Message B
    Message C

              ↓

TCP:
    just a continuous stream of bytes

              ↓

conn.Read():
    "give me whatever bytes are available"
```

So don't build your protocol around:

```go
conn.Read(buf)
```

meaning **"give me one message"**.

Instead, build a function that knows your **message framing**, and have that function call `io.ReadFull` as many times as necessary:

```text
readMessage()
    │
    ├── read header
    │
    ├── determine payload size
    │
    └── read payload
            │
            ├── ReadFull
            ├── ReadFull
            └── ... until complete
```

For a custom TCP protocol, **length-prefix + `io.ReadFull`** is generally the pattern I'd recommend.
