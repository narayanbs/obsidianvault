The key distinction is:

* **TCP has no message boundaries.** Without a length prefix, you need another framing mechanism: EOF, a delimiter, fixed-size records, or a higher-level protocol.
* **UDP already preserves message boundaries.** One `Write`/`Send` corresponds to one datagram, subject to size limits and truncation handling.
* For a production TCP protocol, a **delimiter** (such as `\n`) is usually the simplest alternative to length prefixes.

Below is a production-oriented Go implementation using **newline-delimited TCP** and **datagram-oriented UDP**, with deadlines, cancellation, size limits, partial writes, graceful shutdown, and error handling.

### TCP writer

```go
package main

import (
	"bufio"
	"context"
	"errors"
	"fmt"
	"io"
	"net"
	"time"
)

const (
	tcpAddr       = "127.0.0.1:9000"
	maxMessageLen = 1 << 20 // 1 MiB, including delimiter
	writeTimeout  = 10 * time.Second
)

func writeTCP(ctx context.Context, addr string, messages <-chan []byte) error {
	d := net.Dialer{
		Timeout: 10 * time.Second,
		KeepAlive: 30 * time.Second,
	}

	conn, err := d.DialContext(ctx, "tcp", addr)
	if err != nil {
		return fmt.Errorf("dial TCP %s: %w", addr, err)
	}
	defer conn.Close()

	bw := bufio.NewWriterSize(conn, 32*1024)

	for {
		select {
		case <-ctx.Done():
			return ctx.Err()

		case msg, ok := <-messages:
			if !ok {
				// Closing the connection is the TCP "end of stream"
				// indication when there are no message delimiters left.
				if err := bw.Flush(); err != nil {
					return fmt.Errorf("flush: %w", err)
				}
				return conn.Close()
			}

			if len(msg)+1 > maxMessageLen {
				return fmt.Errorf("message too large: %d bytes", len(msg))
			}

			// Our framing protocol is:
			//
			//     payload + '\n'
			//
			// Therefore payloads may not contain an unescaped '\n'.
			if containsNewline(msg) {
				return errors.New("message contains newline delimiter")
			}

			if err := conn.SetWriteDeadline(time.Now().Add(writeTimeout)); err != nil {
				return fmt.Errorf("set write deadline: %w", err)
			}

			if _, err := bw.Write(msg); err != nil {
				return fmt.Errorf("write message: %w", err)
			}

			if err := bw.WriteByte('\n'); err != nil {
				return fmt.Errorf("write delimiter: %w", err)
			}

			if err := bw.Flush(); err != nil {
				return fmt.Errorf("flush message: %w", err)
			}
		}
	}
}

func containsNewline(b []byte) bool {
	for _, c := range b {
		if c == '\n' {
			return true
		}
	}
	return false
}

func main() {
	ctx := context.Background()

	messages := make(chan []byte)

	go func() {
		defer close(messages)

		messages <- []byte("hello")
		messages <- []byte("second message")
		messages <- []byte("third message")
	}()

	if err := writeTCP(ctx, tcpAddr, messages); err != nil {
		if !errors.Is(err, context.Canceled) {
			fmt.Println("TCP writer:", err)
		}
	}
}
```

### TCP reader

The important part here is **not** calling `Read()` and assuming one `Read()` equals one message. `bufio.Reader.ReadBytes('\n')` handles arbitrary TCP segmentation/coalescing.

```go
package main

import (
	"bufio"
	"context"
	"errors"
	"fmt"
	"io"
	"net"
	"time"
)

const (
	tcpAddr       = "127.0.0.1:9000"
	maxMessageLen = 1 << 20
	idleTimeout   = 5 * time.Minute
)

func readTCP(ctx context.Context, addr string) error {
	d := net.Dialer{
		Timeout:   10 * time.Second,
		KeepAlive: 30 * time.Second,
	}

	conn, err := d.DialContext(ctx, "tcp", addr)
	if err != nil {
		return fmt.Errorf("dial TCP: %w", err)
	}
	defer conn.Close()

	reader := bufio.NewReaderSize(conn, 32*1024)

	for {
		if err := conn.SetReadDeadline(time.Now().Add(idleTimeout)); err != nil {
			return fmt.Errorf("set read deadline: %w", err)
		}

		msg, err := readDelimited(reader, maxMessageLen)
		if err != nil {
			switch {
			case errors.Is(err, io.EOF):
				// Peer cleanly closed its sending side.
				return nil

			case errors.Is(err, context.Canceled):
				return err

			default:
				return fmt.Errorf("read TCP message: %w", err)
			}
		}

		// msg does not contain the '\n' delimiter.
		fmt.Printf("received %d bytes: %q\n", len(msg), msg)
	}
}

func readDelimited(r *bufio.Reader, max int) ([]byte, error) {
	var result []byte

	for {
		part, err := r.ReadSlice('\n')

		if len(part) > 0 {
			if len(result)+len(part) > max {
				return nil, fmt.Errorf("message exceeds %d bytes", max)
			}

			result = append(result, part...)
		}

		if err != nil {
			return nil, err
		}

		if len(result) > 0 && result[len(result)-1] == '\n' {
			result = result[:len(result)-1]
			return result, nil
		}
	}
}

func main() {
	if err := readTCP(context.Background(), "127.0.0.1:9000"); err != nil {
		fmt.Println("TCP reader:", err)
	}
}
```

### UDP writer

UDP is simpler because the datagram itself supplies the boundary. **Do not add a length prefix merely to recover boundaries that UDP already provides.**

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"net"
	"time"
)

const (
	udpAddr      = "127.0.0.1:9001"
	maxUDPPayload = 1200 // conservative application-level limit
)

func writeUDP(ctx context.Context, addr string, messages <-chan []byte) error {
	d := net.Dialer{
		Timeout: 10 * time.Second,
	}

	conn, err := d.DialContext(ctx, "udp", addr)
	if err != nil {
		return fmt.Errorf("dial UDP %s: %w", addr, err)
	}
	defer conn.Close()

	for {
		select {
		case <-ctx.Done():
			return ctx.Err()

		case msg, ok := <-messages:
			if !ok {
				return nil
			}

			if len(msg) > maxUDPPayload {
				return fmt.Errorf(
					"UDP payload too large: %d > %d bytes",
					len(msg),
					maxUDPPayload,
				)
			}

			if err := conn.SetWriteDeadline(time.Now().Add(10 * time.Second)); err != nil {
				return fmt.Errorf("set UDP write deadline: %w", err)
			}

			n, err := conn.Write(msg)
			if err != nil {
				return fmt.Errorf("UDP write: %w", err)
			}

			if n != len(msg) {
				return fmt.Errorf("short UDP write: %d/%d", n, len(msg))
			}
		}
	}
}

func main() {
	ctx := context.Background()

	messages := make(chan []byte)

	go func() {
		defer close(messages)
		messages <- []byte("hello")
		messages <- []byte("another datagram")
	}()

	if err := writeUDP(ctx, udpAddr, messages); err != nil &&
		!errors.Is(err, context.Canceled) {
		fmt.Println("UDP writer:", err)
	}
}
```

### UDP reader

```go
package main

import (
	"context"
	"errors"
	"fmt"
	"net"
	"time"
)

const (
	udpAddr       = "127.0.0.1:9001"
	maxUDPPayload = 64 * 1024
	readTimeout   = 5 * time.Minute
)

func readUDP(ctx context.Context, addr string) error {
	pc, err := net.ListenPacket("udp", addr)
	if err != nil {
		return fmt.Errorf("listen UDP %s: %w", addr, err)
	}
	defer pc.Close()

	buffer := make([]byte, maxUDPPayload)

	for {
		if err := pc.SetReadDeadline(time.Now().Add(readTimeout)); err != nil {
			return fmt.Errorf("set UDP read deadline: %w", err)
		}

		n, remote, err := pc.ReadFrom(buffer)
		if err != nil {
			if ctx.Err() != nil {
				return ctx.Err()
			}

			var netErr net.Error
			if errors.As(err, &netErr) && netErr.Timeout() {
				continue
			}

			return fmt.Errorf("UDP receive: %w", err)
		}

		// n bytes is exactly the datagram payload received.
		msg := buffer[:n]

		fmt.Printf(
			"received %d bytes from %s: %q\n",
			len(msg),
			remote,
			msg,
		)
	}
}

func main() {
	if err := readUDP(context.Background(), udpAddr); err != nil {
		fmt.Println("UDP reader:", err)
	}
}
```

### What "without length prefixes" should mean

For TCP, choose one of these:

| Framing                         | Good for                                                             |
| ------------------------------- | -------------------------------------------------------------------- |
| `message\n`                     | Text/protocol messages                                               |
| HTTP-style headers + blank line | Structured protocols                                                 |
| Fixed-size records              | Binary protocols with known record size                              |
| EOF                             | Exactly one message per connection                                   |
| Escaped delimiter               | Binary/text messages containing arbitrary data                       |
| No framing                      | Only if the application has some other unambiguous semantic boundary |

For **arbitrary binary messages**, a newline delimiter by itself isn't sufficient unless you escape/encode the delimiter. A robust alternative is **COBS, SLIP, or another delimiter-escaping scheme**. Those avoid a length prefix while still giving TCP explicit message boundaries.

For UDP, the natural design is simply:

```text
UDP datagram #1 -> message #1
UDP datagram #2 -> message #2
UDP datagram #3 -> message #3
```

One important production caveat: **UDP does not guarantee delivery, ordering, or uniqueness.** If your application requires those properties, you need application-level mechanisms (sequence numbers, acknowledgements, retransmission, etc.), even though you can still avoid length prefixes.
