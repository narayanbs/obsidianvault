TCP is **stream-oriented**, while **UDP is message-oriented (datagram-oriented)**.

Here's the difference:

| Feature            | TCP                                         | UDP                                |
| ------------------ | ------------------------------------------- | ---------------------------------- |
| Communication type | Stream-oriented                             | Datagram/message-oriented          |
| Data unit          | Byte stream                                 | Datagram (packet)                  |
| Boundaries         | No message boundaries preserved             | Message boundaries preserved       |
| Reliability        | Reliable (acknowledgments, retransmissions) | Unreliable (no delivery guarantee) |
| Ordering           | Guaranteed in order                         | No ordering guarantee              |
| Connection         | Connection-oriented                         | Connectionless                     |

### TCP (Stream-oriented)

TCP treats data as a continuous stream of bytes.

For example, if a sender writes:

```
send("Hello")
send("World")
```

The receiver might receive:

```
HelloWorld
```

or

```
Hel
loWo
rld
```

TCP does **not** preserve the boundaries between `send()` calls. The application must define its own framing (such as using delimiters or length prefixes).

### UDP (Datagram-oriented)

UDP sends independent messages called **datagrams**.

If the sender sends:

```
sendto("Hello")
sendto("World")
```

The receiver gets exactly two separate datagrams:

```
Hello
World
```

Each `recvfrom()` returns one complete datagram (unless the receive buffer is too small, in which case the excess data is discarded). UDP preserves message boundaries.

### Why this matters

* Use **TCP** when you need reliable, ordered delivery (e.g., web browsing, file transfer, email).
* Use **UDP** when low latency is more important than reliability, or when the application can tolerate or handle packet loss (e.g., DNS, VoIP, online gaming, live video streaming).

**In one sentence:**

* **TCP = stream-oriented (continuous byte stream).**
* **UDP = message/datagram-oriented (each packet is a separate message).**
