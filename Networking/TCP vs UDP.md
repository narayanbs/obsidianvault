
# Stream vs Message

### i am trying to understand UDP.. In tcp, we have the handshake afterwards we send streams of bytes.. Its upto the two parties to determine the start and end... in udp , we dont use byte streams, we send messages of a fixed length, the message or datagram has a header that contains the origin, destination details, payload information and the datagram payload is of fixed size , if the payload exceeds, its broken into two datagrams. am i correct?

You're very close, but there are a few important corrections.

### TCP: Byte stream

Your understanding of TCP is correct.

- TCP establishes a connection using the three-way handshake.
- After that, the application sees **one continuous stream of bytes**.
- TCP itself **does not preserve message boundaries**.

For example, if the sender does:
```
send("Hello")
send("World")
```

the receiver might receive:
```
HelloWorld
```

or
```
Hel
loWo
rld
```

The receiving application must decide where one logical message ends and the next begins (using delimiters, length fields, etc.).

### UDP: Message-oriented

This is the major difference.

UDP is **message-oriented**, not stream-oriented.

Each call to `sendto()` sends **one datagram**, and each call to `recvfrom()` receives **one datagram**.

If you send:
```
Datagram 1: "Hello"
Datagram 2: "World"
```

the receiver gets either:
```
Hello
```

and later
```
World
```

or not at all if packets are lost.

They are never merged together like TCP streams.

### Is the payload fixed length?

This is where your understanding needs adjustment.

The payload is **not a fixed size**.

A UDP datagram can have **any payload length** (up to protocol limits).

For example:
```
Datagram 1
------------
Header
Payload = 10 bytes

Datagram 2
------------
Header
Payload = 100 bytes

Datagram 3
------------
Header
Payload = 1000 bytes
```

All of these are perfectly valid.

The UDP header itself is always **8 bytes** long.

---

### What if the payload is too large?

This part is also slightly different.

You said:
> if the payload exceeds, it's broken into two datagrams.

Not exactly.

The application sends **one UDP datagram**.

If that datagram is larger than what the network can transmit in one IP packet, then the **IP layer** (not UDP) may **fragment** it into multiple IP fragments.

For example:

```
Application
     |
     | 5000-byte UDP datagram
     v
UDP
     |
     v
IP
     |
     | Fragment 1
     | Fragment 2
     | Fragment 3
     v
Network

```

The receiver's IP layer attempts to reassemble the fragments before passing the complete UDP datagram to UDP.

If **any fragment is lost**, the entire UDP datagram is discarded.

Because fragmentation is inefficient and fragile, many applications avoid sending UDP datagrams larger than the network's maximum transmission unit (MTU).

---

### Does UDP guarantee delivery?

No.

A UDP datagram may:

- arrive successfully
- arrive out of order
- be duplicated
- be lost completely

UDP provides no acknowledgments, retransmissions, or ordering.

---

### A good mental model

Think of it like this:

**TCP**
```
Application
    |
    |  "ABCDEFGHIJ"
    v
TCP
    |
Network packets
    |
TCP
    v
Application

Receives: ABCDEFGHIJ
```
The packets are invisible to the application.

---

**UDP**
```
Application
    |
    | Message 1
    | Message 2
    | Message 3
    v
UDP

Datagram 1
Datagram 2
Datagram 3

Receiver gets exactly those individual messages
(or loses some of them).
```
Each datagram is an independent message.

---

So, with a few refinements, your understanding becomes:

- ✅ TCP uses a handshake and provides a continuous byte stream with no message boundaries.
- ✅ UDP is message-oriented: each send corresponds to one datagram.
- ❌ UDP datagrams do **not** have a fixed payload size; the payload length can vary.
- ❌ If a datagram is too large, UDP itself does **not** split it into multiple UDP datagrams. Instead, the IP layer may fragment the underlying IP packet, or the application can choose to send multiple separate UDP datagrams.


## Examples in Golang

f you're learning UDP networking in Go, it's helpful to understand the basic flow first:

- **Server:** Listen on a UDP port → receive datagrams → optionally send a response.
- **Client:** Send a datagram to the server → optionally wait for a reply.

Here's a minimal tutorial.
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	addr, err := net.ResolveUDPAddr("udp", ":8080")
	if err != nil {
		panic(err)
	}

	conn, err := net.ListenUDP("udp", addr)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	fmt.Println("UDP server listening on :8080")

	buffer := make([]byte, 1024)

	for {
		n, clientAddr, err := conn.ReadFromUDP(buffer)
		if err != nil {
			fmt.Println("Read error:", err)
			continue
		}

		message := string(buffer[:n])
		fmt.Printf("Received from %v: %s\n", clientAddr, message)

		reply := []byte("Hello from server!")
		_, err = conn.WriteToUDP(reply, clientAddr)
		if err != nil {
			fmt.Println("Write error:", err)
		}
	}
}
```
## UDP Client
```go
package main

import (
	"fmt"
	"net"
)

func main() {
	serverAddr, err := net.ResolveUDPAddr("udp", "127.0.0.1:8080")
	if err != nil {
		panic(err)
	}

	conn, err := net.DialUDP("udp", nil, serverAddr)
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	_, err = conn.Write([]byte("Hello, server!"))
	if err != nil {
		panic(err)
	}

	buffer := make([]byte, 1024)
	n, err := conn.Read(buffer)
	if err != nil {
		panic(err)
	}

	fmt.Println("Server replied:", string(buffer[:n]))
}
```
## Running

Start the server:
```go
go run server.go
```
In another terminal:
```go
go run client.go
```
**Server**
```go
UDP server listening on :8080
Received from 127.0.0.1:54321: Hello, server!
```
**Client**
```go
Server replied: Hello from server!
```
## How UDP Works

Unlike TCP:

- No connection handshake.
- Messages (datagrams) are independent.
- Packets can be lost, duplicated, or arrive out of order.
- Lower overhead and lower latency.

Typical uses include:

- DNS
- Online games
- VoIP
- Video streaming
- Service discovery

## Key API Functions

|Function|Purpose|
|---|---|
|`net.ResolveUDPAddr()`|Parses a UDP address.|
|`net.ListenUDP()`|Creates a UDP server socket.|
|`net.DialUDP()`|Creates a UDP client connected to a remote endpoint.|
|`ReadFromUDP()`|Receives a packet and the sender's address.|
|`WriteToUDP()`|Sends a packet to a specific address.|
|`Read()`|Reads from a connected UDP socket.|
|`Write()`|Writes to a connected UDP socket.|

## Setting Read Deadlines

To avoid blocking forever:
```go
conn.SetReadDeadline(time.Now().Add(5 * time.Second))

buf := make([]byte, 1024)
n, err := conn.Read(buf)
if err != nil {
	if netErr, ok := err.(net.Error); ok && netErr.Timeout() {
		fmt.Println("Read timed out")
	}
}
```
## Concurrent Server

A simple way to process packets concurrently:
```go
for {
	buf := make([]byte, 1024)
	n, addr, err := conn.ReadFromUDP(buf)
	if err != nil {
		continue
	}

	data := append([]byte(nil), buf[:n]...)

	go func(addr *net.UDPAddr, msg []byte) {
		fmt.Printf("%s\n", msg)
		conn.WriteToUDP([]byte("OK"), addr)
	}(addr, data)
}

```
Notice that the received bytes are copied (`append([]byte(nil), buf[:n]...)`) before starting the goroutine. Without this, the next `ReadFromUDP` call could overwrite the buffer while the goroutine is still using it.

## Best Practices

- Reuse buffers when possible to reduce allocations.
- Set read/write deadlines for production services.
- Validate packet sizes and contents.
- Keep UDP packets reasonably small (typically below the network MTU, around 1200–1400 bytes for internet traffic) to reduce fragmentation.
- Build your own retry, sequencing, or acknowledgment logic if your application requires reliable delivery.

Once you're comfortable with these basics, you can explore more advanced topics such as multicast, broadcasting, and high-performance packet processing using Go's networking packages.
