# Reader
 Assuming you want a **length-prefixed TCP protocol**, here's a solid production-oriented Go implementation.

The wire format is:

```text
┌──────────────┬──────────────────────┐
│ 4-byte uint32│      payload         │
│ length       │      N bytes         │
└──────────────┴──────────────────────┘
```

So a message `"hello"` becomes:

```text
00 00 00 05 68 65 6c 6c 6f
```

### Production-oriented reader

```go
package main

import (
	"encoding/binary"
	"errors"
	"fmt"
	"io"
	"log"
	"net"
	"time"
)

const (
	maxMessageSize = 10 * 1024 * 1024 // 10 MiB
	headerSize     = 4

	readTimeout = 30 * time.Second
)

func readMessage(conn net.Conn) ([]byte, error) {
	// Prevent a peer from keeping this connection blocked forever.
	if err := conn.SetReadDeadline(time.Now().Add(readTimeout)); err != nil {
		return nil, fmt.Errorf("set read deadline: %w", err)
	}

	// Read exactly the 4-byte length header.
	var header [headerSize]byte

	if _, err := io.ReadFull(conn, header[:]); err != nil {
		if errors.Is(err, io.EOF) {
			return nil, io.EOF
		}

		if errors.Is(err, io.ErrUnexpectedEOF) {
			return nil, fmt.Errorf("incomplete message header: %w", err)
		}

		return nil, fmt.Errorf("read message header: %w", err)
	}

	// Decode payload length.
	length := binary.BigEndian.Uint32(header[:])

	// Never trust a length supplied by the peer.
	if length > maxMessageSize {
		return nil, fmt.Errorf(
			"message too large: %d bytes (max %d)",
			length,
			maxMessageSize,
		)
	}

	// Zero-length messages are allowed here.
	if length == 0 {
		return []byte{}, nil
	}

	// Read exactly `length` bytes.
	payload := make([]byte, length)

	if _, err := io.ReadFull(conn, payload); err != nil {
		if errors.Is(err, io.EOF) || errors.Is(err, io.ErrUnexpectedEOF) {
			return nil, fmt.Errorf(
				"incomplete message body: expected %d bytes: %w",
				length,
				err,
			)
		}

		return nil, fmt.Errorf("read message body: %w", err)
	}

	return payload, nil
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	for {
		message, err := readMessage(conn)
		if err != nil {
			if errors.Is(err, io.EOF) {
				log.Println("client disconnected")
				return
			}

			log.Printf("connection error: %v", err)
			return
		}

		log.Printf("received %d bytes: %q", len(message), message)

		// Process the message here.
	}
}

func main() {
	listener, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatal(err)
	}
	defer listener.Close()

	log.Println("listening on :8080")

	for {
		conn, err := listener.Accept()
		if err != nil {
			// In a real server, you may want to distinguish
			// temporary errors from permanent ones.
			log.Printf("accept error: %v", err)
			continue
		}

		log.Printf("client connected: %s", conn.RemoteAddr())

		go handleConnection(conn)
	}
}
```

### Why these pieces matter

The important part is:

```go
io.ReadFull(conn, header[:])
```

followed by:

```go
io.ReadFull(conn, payload)
```

Suppose the sender writes:

```text
[length][payload]
```

but TCP delivers it as:

```text
Read #1:  [length][first 100 bytes]
Read #2:  [next 500 bytes]
Read #3:  [remaining bytes]
```

You don't care. `io.ReadFull` keeps reading until the requested number of bytes has arrived.

It also handles the opposite situation:

```text
Read #1:
[length][payload1][payload2][payload3]
```

The first `io.ReadFull` consumes only the bytes needed for the current operation. The next call reads the next message. **You don't need to manually worry about TCP packet boundaries.**

### One thing I'd change for a real production protocol

If this is going beyond a toy/internal service, I'd define the protocol explicitly, for example:

```text
+----------------+------------------+
| Length (4)     | Payload (N)      |
+----------------+------------------+
   uint32 BE          N bytes
```

Then enforce things like:

* maximum message size
* read deadlines
* write deadlines
* connection limits
* graceful shutdown
* authentication/TLS if applicable
* protocol/version field if the protocol may evolve
* structured error handling/logging

And **don't allocate based on an untrusted length before checking the maximum**, which is why the example checks `maxMessageSize` first.

If you're building a **request/response TCP server**, I can also show you the full production pattern including **framing + writes + deadlines + graceful shutdown + connection cancellation**.

--------
# Writer

For the same **4-byte big-endian length-prefixed protocol**, the writer should be the mirror image of the reader.

The important production detail is: **don't assume one `conn.Write()` writes everything**. `net.Conn.Write` normally returns the number of bytes written, and you should handle short writes.

```go
package main

import (
	"encoding/binary"
	"fmt"
	"io"
	"net"
	"time"
)

const (
	maxMessageSize = 10 * 1024 * 1024 // 10 MiB
	writeTimeout   = 30 * time.Second
)

// writeMessage writes:
//
//	[4-byte payload length][payload]
//
// Length is encoded as big-endian uint32.
func writeMessage(conn net.Conn, payload []byte) error {
	// Validate before constructing/sending anything.
	if len(payload) > maxMessageSize {
		return fmt.Errorf(
			"message too large: %d bytes (max %d)",
			len(payload),
			maxMessageSize,
		)
	}

	if len(payload) > int(^uint32(0)) {
		return fmt.Errorf("message too large for uint32 length prefix")
	}

	// Prevent a broken/unresponsive peer from blocking forever.
	if err := conn.SetWriteDeadline(time.Now().Add(writeTimeout)); err != nil {
		return fmt.Errorf("set write deadline: %w", err)
	}

	// Construct the 4-byte length header.
	var header [4]byte
	binary.BigEndian.PutUint32(header[:], uint32(len(payload)))

	// Write the header completely.
	if err := writeFull(conn, header[:]); err != nil {
		return fmt.Errorf("write message header: %w", err)
	}

	// Write the payload completely.
	if err := writeFull(conn, payload); err != nil {
		return fmt.Errorf("write message payload: %w", err)
	}

	return nil
}

// writeFull keeps writing until the entire buffer has been written.
func writeFull(w io.Writer, data []byte) error {
	for len(data) > 0 {
		n, err := w.Write(data)

		if n > 0 {
			data = data[n:]
		}

		if err != nil {
			return err
		}

		if n == 0 {
			return io.ErrShortWrite
		}
	}

	return nil
}
```

### But there's an important concurrency issue

For a real TCP server, **don't allow multiple goroutines to call `writeMessage` on the same connection concurrently**.

For example, this is dangerous:

```go
go writeMessage(conn, []byte("hello"))
go writeMessage(conn, []byte("world"))
```

You could end up with the bytes from the two messages interleaved at the application-protocol level:

```text
[length of hello][length of world][hello][world]
```

or other undesirable ordering.

Instead, serialize writes.

A simple production-oriented connection wrapper:

```go
type Client struct {
	conn net.Conn

	writeMu sync.Mutex
}

func (c *Client) WriteMessage(payload []byte) error {
	c.writeMu.Lock()
	defer c.writeMu.Unlock()

	return writeMessage(c.conn, payload)
}
```

You'll need:

```go
import "sync"
```

Then:

```go
client.WriteMessage([]byte("hello"))
```

and:

```go
go client.WriteMessage([]byte("hello"))
go client.WriteMessage([]byte("world"))
go client.WriteMessage([]byte("foo"))
```

are safe from **concurrent writers corrupting the framing**, although the order in which those goroutines acquire the mutex is not guaranteed.

### One improvement I'd make

For a higher-throughput server, I would actually **combine the header and payload into one buffer** and perform one `writeFull`:

```go
func writeMessage(conn net.Conn, payload []byte) error {
	if len(payload) > maxMessageSize {
		return fmt.Errorf("message too large: %d", len(payload))
	}

	if err := conn.SetWriteDeadline(time.Now().Add(writeTimeout)); err != nil {
		return fmt.Errorf("set write deadline: %w", err)
	}

	message := make([]byte, 4+len(payload))

	binary.BigEndian.PutUint32(
		message[:4],
		uint32(len(payload)),
	)

	copy(message[4:], payload)

	if err := writeFull(conn, message); err != nil {
		return fmt.Errorf("write message: %w", err)
	}

	return nil
}
```

This has a nice property:

```text
writeFull()
    │
    └── [length][payload]
             │
             ▼
          TCP stream
```

TCP still might split it into multiple packets/writes internally, **but your application framing remains intact**.

For a serious implementation, I'd use the second version + a `writeMu` (or, even better, a dedicated per-connection writer goroutine/channel).
