
The **`net/netip`** package, introduced in **Go 1.18**, represents a modern rework of IP address primitives in Go.

Before `netip`, Go relied on `net.IP`, which suffered from major performance and API drawbacks. `net/netip` was designed to fix these limitations by offering **small, immutable, allocation-free, and comparable** network address types.

### Why standard `net.IP` needed a replacement

| **Feature**           | **net.IP (Legacy)**                    | **net/netip.Addr (Modern)**                   |
| --------------------- | -------------------------------------- | --------------------------------------------- |
| **Data Structure**    | Slice (`[]byte`)                       | Small value struct (128-bit int + zone)       |
| **Memory Allocation** | Heap allocated (generates GC pressure) | Stack allocated (zero allocations)            |
| **Mutability**        | **Mutable** (prone to side-effects)    | **Immutable** (safe to share/pass by value)   |
| **Comparability**     | **No** (Slices cannot use `==`)        | **Yes** (Supports `==`, usable as `map` keys) |
| **Zone IDs**          | Handled separately via `net.IPAddr`    | Natively supported in the core struct         |
## Core Types in `net/netip`

The package revolves around three core primitives:

### 1. `netip.Addr`

Represents a single IPv4 or IPv6 address (with optional IPv6 zone IDs)

```go
package main

import (
	"fmt"
	"net/netip"
)

func main() {
	// Parsing
	ip, err := netip.ParseAddr("192.0.2.1")
	if err != nil {
		panic(err)
	}

	// Properties
	fmt.Println("Is IPv4:", ip.Is4())           // true
	fmt.Println("Is IPv6:", ip.Is6())           // false
	fmt.Println("Is Private:", ip.IsPrivate())   // true
	fmt.Println("Is Loopback:", ip.IsLoopback()) // false

	// Direct Equality & Map Usage
	ip2 := netip.MustParseAddr("192.0.2.1")
	fmt.Println("Equal:", ip == ip2) // true

	ipMap := map[netip.Addr]string{
		ip: "Gateway Router",
	}
	fmt.Println(ipMap[ip]) // Gateway Router
}
```

### 2. `netip.AddrPort`

Represents an IP address paired with a port number (`Addr` + `uint16`). It eliminates redundant string concats and manual string parsing.

```go
ap, err := netip.ParseAddrPort("10.0.0.1:8080")
if err != nil {
	panic(err)
}

fmt.Println(ap.Addr()) // 10.0.0.1
fmt.Println(ap.Port()) // 8080

// Programmatic Construction
addr := netip.MustParseAddr("::1")
ap2 := netip.AddrPortFrom(addr, 443)
fmt.Println(ap2.String()) // [::1]:443
```

### 3. `netip.Prefix`

Represents an IP network prefix using CIDR notation (an `Addr` paired with a prefix bit length).

```go
prefix, err := netip.ParsePrefix("192.168.1.0/24")
if err != nil {
	panic(err)
}

ip1 := netip.MustParseAddr("192.168.1.50")
ip2 := netip.MustParseAddr("10.0.0.1")

// Checking containment
fmt.Println(prefix.Contains(ip1)) // true
fmt.Println(prefix.Contains(ip2)) // false

// Subnet properties
fmt.Println(prefix.Masked()) // Normalizes the prefix if host bits are set
```

## Notable Features & Tricks

### 1. Pointer-Free IP Iteration

Because `Addr` is arithmetic under the hood, you can step forward or backward through IP addresses easily:

```go
ip := netip.MustParseAddr("192.168.1.1")
nextIP := ip.Next() // 192.168.1.2
prevIP := ip.Prev() // 192.168.1.0
```

### 2. `MustParse*` Helper Functions

For static/hardcoded constants, use `MustParseAddr`, `MustParseAddrPort`, or `MustParsePrefix` to eliminate boilerplate error handling (they panic on invalid inputs, ideal for package-level variable declarations).

```go
var LoopbackLocal = netip.MustParseAddr("127.0.0.1")
```

### 3. Converting to and from Legacy `net.IP`

When working with legacy Go packages or stdlib network interfaces (like `net.Dial`), convert back and forth easily:

```go
// From net.IP to netip.Addr
stdIP := net.ParseIP("192.0.2.1")
addr, ok := netip.AddrFromSlice(stdIP)
if ok {
    // Standard net.ParseIP returns IPv4 addresses mapped as 16-byte IPv6.
    // Call Unmap() to extract pure IPv4 if needed.
    addr = addr.Unmap()
}

// From netip.Addr to []byte / net.IP
sliceIP := addr.AsSlice() // Returns []byte
legacyIP := net.IP(sliceIP)
```

### 4.  Print all hosts in a network

For example, for 192.168.1.0/24, you can iterate over the addresses in the prefix:

```go
package main

import (
	"fmt"
	"net/netip"
)

func main() {
	prefix := netip.MustParsePrefix("192.168.1.0/24")

	addr := prefix.Addr()
	for prefix.Contains(addr) {
		fmt.Println(addr)
		addr = addr.Next()
	}
}
```

Excluding network and broadcast addresses
```go
prefix := netip.MustParsePrefix("192.168.1.0/24")

first := prefix.Addr().Next()
last := prefix.Broadcast()

for addr := first; addr != last; addr = addr.Next() {
	fmt.Println(addr)
}
```

## Key Takeaways

1. **Default Choice:** Use `net/netip` for any new Go project handling IP routing, rate-limiting, firewalling, or networking calculations.
    
2. **Performance:** Offers **0 heap allocations** for address manipulation, resulting in lower GC latency in high-throughput network applications.
    
3. **Map Keys:** Unlocks direct use of IPs and subnets in maps, set structures, and hash sets natively.


# Detailed Notes


## 1. What is `net/netip`?

The `net/netip` package was introduced to provide modern IP address types in Go.

The main types you'll use are:

* `netip.Addr` — an IPv4 or IPv6 address
* `netip.Prefix` — an IP network/prefix such as `192.168.1.0/24`
* `netip.AddrPort` — an IP address + port, such as `192.168.1.10:8080`

Import it with:

```go
package main

import (
	"fmt"
	"net/netip"
)

func main() {
	addr := netip.MustParseAddr("192.168.1.10")

	fmt.Println(addr)
}
```

Output:

```text
192.168.1.10
```

---

# 2. `netip.Addr`

The most important type is `netip.Addr`.

Think of it as:

```text
netip.Addr = an IP address
```

For example:

```go
addr := netip.MustParseAddr("192.168.1.10")
```

You can also parse IPv6:

```go
addr := netip.MustParseAddr("2001:db8::1")

fmt.Println(addr)
```

Output:

```text
2001:db8::1
```

### `ParseAddr` vs `MustParseAddr`

There are two common ways to parse an address.

### `ParseAddr`

Returns an error:

```go
addr, err := netip.ParseAddr("192.168.1.10")
if err != nil {
	fmt.Println("invalid IP:", err)
	return
}

fmt.Println(addr)
```

This is appropriate when the IP comes from a user, network request, configuration file, etc.

### `MustParseAddr`

Panics if the address is invalid:

```go
addr := netip.MustParseAddr("192.168.1.10")
```

This is convenient for constants that you know are valid.

For example:

```go
var localhost = netip.MustParseAddr("127.0.0.1")
```

Don't normally use `MustParseAddr` on untrusted input.

---

# 3. Checking whether an address is IPv4 or IPv6

```go
addr := netip.MustParseAddr("192.168.1.10")

fmt.Println(addr.Is4())
fmt.Println(addr.Is6())
```

Output:

```text
true
false
```

For IPv6:

```go
addr := netip.MustParseAddr("2001:db8::1")

fmt.Println(addr.Is4())
fmt.Println(addr.Is6())
```

Output:

```text
false
true
```

You can also check whether an address is valid:

```go
addr, err := netip.ParseAddr("hello")

if err != nil {
	fmt.Println("invalid address")
}
```

---

# 4. Special IP addresses

`netip.Addr` has useful methods for identifying special addresses.

For example:

```go
addr := netip.MustParseAddr("127.0.0.1")

fmt.Println(addr.IsLoopback())
```

Output:

```text
true
```

Some useful methods include:

```go
addr.IsLoopback()
addr.IsPrivate()
addr.IsUnspecified()
addr.IsMulticast()
addr.IsGlobalUnicast()
```

For example:

```go
addresses := []string{
	"127.0.0.1",
	"192.168.1.10",
	"8.8.8.8",
}

for _, s := range addresses {
	addr := netip.MustParseAddr(s)

	fmt.Println(
		addr,
		"private:", addr.IsPrivate(),
		"loopback:", addr.IsLoopback(),
	)
}
```

This is particularly useful when writing network access-control logic.

---

# 5. Comparing IP addresses

`netip.Addr` can be compared directly.

```go
a := netip.MustParseAddr("192.168.1.10")
b := netip.MustParseAddr("192.168.1.20")

fmt.Println(a == b)
```

Output:

```text
false
```

You can also use:

```go
a.Compare(b)
```

The result is:

* `-1` → `a < b`
* `0` → equal
* `1` → `a > b`

Example:

```go
a := netip.MustParseAddr("192.168.1.10")
b := netip.MustParseAddr("192.168.1.20")

fmt.Println(a.Compare(b))
```

---

# 6. `netip.Prefix`

Now we get to something much more interesting.

A prefix represents a network:

```text
192.168.1.0/24
```

You can create one like this:

```go
prefix := netip.MustParsePrefix("192.168.1.0/24")

fmt.Println(prefix)
```

Think of:

```text
192.168.1.0/24
```

as:

```text
network address = 192.168.1.0
prefix length   = 24
```

IPv6 works too:

```go
prefix := netip.MustParsePrefix("2001:db8::/32")
```

---

# 7. Checking whether an IP belongs to a network

This is probably one of the most useful features of `netip`.

Suppose you have:

```text
192.168.1.0/24
```

and want to know whether:

```text
192.168.1.42
```

belongs to that network.

Use:

```go
network := netip.MustParsePrefix("192.168.1.0/24")
addr := netip.MustParseAddr("192.168.1.42")

fmt.Println(network.Contains(addr))
```

Output:

```text
true
```

Try another:

```go
addr := netip.MustParseAddr("192.168.2.42")

fmt.Println(network.Contains(addr))
```

Output:

```text
false
```

This is extremely useful for things like:

* IP allowlists
* firewall rules
* subnet checks
* routing logic
* private-network detection
* access control

For example:

```go
func allowed(addr netip.Addr) bool {
	network := netip.MustParsePrefix("10.0.0.0/8")

	return network.Contains(addr)
}
```

Then:

```go
fmt.Println(allowed(netip.MustParseAddr("10.20.30.40")))
fmt.Println(allowed(netip.MustParseAddr("192.168.1.10")))
```

---

# 8. Prefix length

You can get the prefix length using:

```go
prefix := netip.MustParsePrefix("192.168.1.0/24")

bits := prefix.Bits()

fmt.Println(bits)
```

Output:

```text
24
```

For IPv6:

```go
prefix := netip.MustParsePrefix("2001:db8::/32")

fmt.Println(prefix.Bits())
```

Output:

```text
32
```

---

# 9. Getting the network address

Suppose somebody gives you:

```text
192.168.1.123/24
```

The actual network is:

```text
192.168.1.0/24
```

You can use:

```go
prefix := netip.MustParsePrefix("192.168.1.123/24")

network := prefix.Masked()

fmt.Println(network)
```

Output:

```text
192.168.1.0/24
```

This is an important distinction:

```go
prefix := netip.MustParsePrefix("192.168.1.123/24")

fmt.Println(prefix)
fmt.Println(prefix.Masked())
```

Output:

```text
192.168.1.123/24
192.168.1.0/24
```

`Masked()` applies the network mask.

---

# 10. `netip.AddrPort`

Another useful type is:

```go
netip.AddrPort
```

It represents:

```text
IP address + port
```

For example:

```text
192.168.1.10:8080
```

You can parse it:

```go
addrPort, err := netip.ParseAddrPort("192.168.1.10:8080")

if err != nil {
	fmt.Println(err)
	return
}

fmt.Println(addrPort)
```

You can access the components:

```go
fmt.Println(addrPort.Addr())
fmt.Println(addrPort.Port())
```

Output:

```text
192.168.1.10
8080
```

IPv6 is especially convenient because it handles brackets correctly:

```go
addrPort, err := netip.ParseAddrPort("[2001:db8::1]:8080")

if err != nil {
	panic(err)
}

fmt.Println(addrPort.Addr())
fmt.Println(addrPort.Port())
```

---

# 11. Creating an `AddrPort`

You don't have to parse a string.

You can construct one:

```go
addr := netip.MustParseAddr("192.168.1.10")

endpoint := netip.AddrPortFrom(addr, 8080)

fmt.Println(endpoint)
```

Output:

```text
192.168.1.10:8080
```

This can be useful when building network clients or servers.

---

# 12. Converting `netip` to strings

All three major types have sensible string representations.

```go
addr := netip.MustParseAddr("192.168.1.10")
prefix := netip.MustParsePrefix("192.168.1.0/24")
addrPort := netip.MustParseAddrPort("192.168.1.10:8080")

fmt.Println(addr.String())
fmt.Println(prefix.String())
fmt.Println(addrPort.String())
```

Output:

```text
192.168.1.10
192.168.1.0/24
192.168.1.10:8080
```

You can also use them with `fmt`:

```go
fmt.Printf("IP: %v\n", addr)
```

---

# 13. Why use `netip` instead of `net.IP`?

You'll often encounter the older:

```go
net.IP
```

from the `net` package.

For example:

```go
ip := net.ParseIP("192.168.1.10")
```

Modern Go code can often use:

```go
addr, err := netip.ParseAddr("192.168.1.10")
```

There are several advantages to `netip`.

### `netip.Addr` is a value type

You can treat it much more like a normal value:

```go
a := netip.MustParseAddr("10.0.0.1")
b := a

fmt.Println(a == b)
```

With `net.IP`, things are different because `net.IP` is a byte slice.

### `netip` has explicit address semantics

Instead of manipulating byte slices, you get methods such as:

```go
addr.Is4()
addr.Is6()
addr.IsPrivate()
addr.IsLoopback()
```

### Prefix handling is straightforward

```go
prefix.Contains(addr)
prefix.Bits()
prefix.Masked()
```

This makes subnet-related code much cleaner.

---

# 14. A practical example: IP allowlist

Imagine you're writing an HTTP server and only want clients from:

```text
10.0.0.0/8
192.168.0.0/16
```

You could write:

```go
package main

import (
	"fmt"
	"net/netip"
)

var allowedNetworks = []netip.Prefix{
	netip.MustParsePrefix("10.0.0.0/8"),
	netip.MustParsePrefix("192.168.0.0/16"),
}

func allowed(addr netip.Addr) bool {
	for _, network := range allowedNetworks {
		if network.Contains(addr) {
			return true
		}
	}

	return false
}

func main() {
	tests := []string{
		"10.20.30.40",
		"192.168.1.100",
		"8.8.8.8",
	}

	for _, s := range tests {
		addr := netip.MustParseAddr(s)

		fmt.Println(s, allowed(addr))
	}
}
```

Output:

```text
10.20.30.40 true
192.168.1.100 true
8.8.8.8 false
```

That's a very realistic use of `netip`.

---

# 15. Parsing an IP from an HTTP request

Here's another realistic example.

Suppose you're dealing with an HTTP server:

```go
func checkIP(ipString string) bool {
	addr, err := netip.ParseAddr(ipString)
	if err != nil {
		return false
	}

	privateNetwork := netip.MustParsePrefix("10.0.0.0/8")

	return privateNetwork.Contains(addr)
}
```

Then:

```go
if checkIP("10.1.2.3") {
	fmt.Println("allowed")
}
```

One important real-world issue: **don't blindly trust an HTTP `X-Forwarded-For` header as the client's IP**. If your server sits behind a reverse proxy, you need to know which proxy is trusted and how it communicates the original client address.

---

# 16. Useful methods to remember

If you're learning `netip`, I would memorize these first.

### `netip.Addr`

```go
netip.ParseAddr(...)
netip.MustParseAddr(...)
addr.Is4()
addr.Is6()
addr.IsPrivate()
addr.IsLoopback()
addr.IsMulticast()
addr.IsUnspecified()
addr.IsGlobalUnicast()
addr.String()
```

### `netip.Prefix`

```go
netip.ParsePrefix(...)
netip.MustParsePrefix(...)
prefix.Contains(addr)
prefix.Bits()
prefix.Masked()
prefix.String()
```

### `netip.AddrPort`

```go
netip.ParseAddrPort(...)
netip.AddrPortFrom(...)
addrPort.Addr()
addrPort.Port()
addrPort.String()
```

---

# 17. A mental model

A good way to remember the package is:

```text
             net/netip
                 │
       ┌─────────┼─────────┐
       │         │         │
     Addr      Prefix    AddrPort
       │         │         │
       │         │         │
      IP       Network   IP + port
       │         │         │
  192.168.1.5  192.168.1.0/24  192.168.1.5:8080
```

And the most common relationship is:

```text
Addr ────────── belongs to ──────────> Prefix

192.168.1.42                     192.168.1.0/24
      │                                  │
      └──────── prefix.Contains(addr) ───┘
                         ↓
                       true
```

---

## 18. A small exercise

Try implementing this function:

```go
func isAllowed(ip string) bool
```

It should return `true` when the IP belongs to either:

```text
10.0.0.0/8
192.168.0.0/16
172.16.0.0/12
```

and `false` otherwise.

For example:

```text
10.20.30.40       → true
192.168.1.50      → true
172.20.1.5        → true
8.8.8.8           → false
```

The important pattern to learn is:

```go
addr, err := netip.ParseAddr(ip)
if err != nil {
	return false
}

for _, network := range networks {
	if network.Contains(addr) {
		return true
	}
}

return false
```

Once you're comfortable with `Addr`, `Prefix`, and `Contains`, you've got most of the core `netip` API down.


## `net.IP` → `netip.Addr`

Use `netip.AddrFromSlice`:

```go
package main

import (
	"fmt"
	"net"
	"net/netip"
)

func main() {
	ip := net.ParseIP("192.168.1.10")

	addr, ok := netip.AddrFromSlice(ip)
	if !ok {
		panic("invalid IP")
	}

	fmt.Println(addr)
}
```

Output:

```text
192.168.1.10
```

### Important detail

`AddrFromSlice` returns:

```go
netip.Addr, bool
```

The `bool` tells you whether the byte slice represented a valid IP address.

---

## `netip.Addr` → `net.IP`

Use `AsSlice()`:

```go
package main

import (
	"fmt"
	"net"
	"net/netip"
)

func main() {
	addr := netip.MustParseAddr("192.168.1.10")

	ip := net.IP(addr.AsSlice())

	fmt.Println(ip)
}
```

Output:

```text
192.168.1.10
```

So the basic conversion is:

```text
net.IP              netip.Addr
  │                     │
  │ AddrFromSlice()     │
  ├────────────────────>│
  │                     │
  │                     │ AsSlice()
  │<────────────────────┤
  │
```

## IPv6 works too

```go
addr := netip.MustParseAddr("2001:db8::1")

ip := net.IP(addr.AsSlice())

fmt.Println(ip)
```

And the reverse:

```go
ip := net.ParseIP("2001:db8::1")

addr, ok := netip.AddrFromSlice(ip)
if !ok {
	panic("invalid IP")
}

fmt.Println(addr)
```

---

## One thing to watch out for

`net.IP` is a `[]byte`, while `netip.Addr` is a value type. So this:

```go
ip := net.IP(addr.AsSlice())
```

creates a `net.IP` backed by the slice returned by `AsSlice()`.

If you're passing it around normally, that's generally fine. If you need an independent mutable byte slice, make a copy:

```go
ip := net.IP(append([]byte(nil), addr.AsSlice()...))
```

### Quick cheat sheet

```go
// net.IP -> netip.Addr
addr, ok := netip.AddrFromSlice(ip)

// netip.Addr -> net.IP
ip := net.IP(addr.AsSlice())
```

There's also a useful distinction between **`net.IP`**, **`netip.Addr`**, and **`net.IPNet`/`netip.Prefix`** that becomes important when converting subnet/routing code.


If you're learning Go networking, the **next useful step** is understanding how `net/netip` works together with `net`, `net/http`, TCP/UDP sockets, and `net/netip.AddrPort`.

The key is to understand that **`netip` doesn't replace `net`**. They work together:

```text
netip
 ├── Addr       → IP address
 ├── Prefix     → IP network/subnet
 └── AddrPort   → IP + port
          │
          ▼
net
 ├── TCP connections
 ├── UDP connections
 ├── listeners
 └── DNS
          │
          ▼
net/http
 └── HTTP servers and clients
```

Let's connect them together with practical examples.

---

# 1. `netip.Addr` and TCP

Suppose you want to connect to:

```text
192.168.1.10:8080
```

With `netip`, you can represent the endpoint cleanly:

```go
package main

import (
	"fmt"
	"net"
	"net/netip"
)

func main() {
	addr := netip.MustParseAddr("192.168.1.10")
	addrPort := netip.AddrPortFrom(addr, 8080)

	fmt.Println(addrPort)

	conn, err := net.Dial("tcp", addrPort.String())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	fmt.Println("connected:", conn.RemoteAddr())
}
```

The important transition is:

```text
netip.AddrPort
      │
      │ String()
      ▼
"192.168.1.10:8080"
      │
      ▼
net.Dial()
```

`net.Dial` traditionally accepts a string address, so `AddrPort.String()` is a convenient bridge.

---

# 2. But `net.Dial` already gives you `net.Addr`

After connecting:

```go
conn, err := net.Dial("tcp", "192.168.1.10:8080")
```

you can do:

```go
remote := conn.RemoteAddr()

fmt.Println(remote)
fmt.Println(remote.Network())
```

For TCP, `RemoteAddr()` typically gives you a `net.Addr`, commonly implemented by `*net.TCPAddr`.

You can type-assert it:

```go
tcpAddr, ok := conn.RemoteAddr().(*net.TCPAddr)
if !ok {
	panic("not a TCP address")
}

fmt.Println(tcpAddr.IP)
fmt.Println(tcpAddr.Port)
```

Here you're back in the older `net` world:

```text
net.TCPAddr
 ├── IP   → net.IP
 └── Port → int
```

If you want `netip`:

```go
addr, ok := netip.AddrFromSlice(tcpAddr.IP)
if !ok {
	panic("invalid IP")
}

addrPort := netip.AddrPortFrom(addr, uint16(tcpAddr.Port))

fmt.Println(addrPort)
```

So you can move between the APIs.

---

# 3. `netip.AddrPort` is particularly useful for TCP/UDP

An `AddrPort` represents:

```text
IP + port
```

For example:

```go
addrPort := netip.MustParseAddrPort("127.0.0.1:8080")
```

You can access both pieces:

```go
fmt.Println(addrPort.Addr())
fmt.Println(addrPort.Port())
```

Output:

```text
127.0.0.1
8080
```

This is useful because you don't have to manually deal with IPv4 vs IPv6 formatting.

For example:

```go
addrPort := netip.MustParseAddrPort("[::1]:8080")

fmt.Println(addrPort.Addr())
fmt.Println(addrPort.Port())
```

Output:

```text
::1
8080
```

---

# 4. TCP server

Now let's use `net` and `netip` together to build a TCP server.

```go
package main

import (
	"fmt"
	"net"
	"net/netip"
)

func main() {
	addr := netip.MustParseAddr("127.0.0.1")
	addrPort := netip.AddrPortFrom(addr, 8080)

	listener, err := net.Listen("tcp", addrPort.String())
	if err != nil {
		panic(err)
	}
	defer listener.Close()

	fmt.Println("listening on", listener.Addr())

	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("accept error:", err)
			continue
		}

		go handle(conn)
	}
}

func handle(conn net.Conn) {
	defer conn.Close()

	fmt.Println("client:", conn.RemoteAddr())

	conn.Write([]byte("Hello from server\n"))
}
```

Notice the division of responsibilities:

```text
netip
  ↓
construct the address

net
  ↓
create the listener
accept TCP connections
read/write data
```

That's a good general pattern.

---

# 5. Getting the client's IP with `netip`

This is where `netip` becomes especially nice.

Suppose:

```go
remote := conn.RemoteAddr()
```

For a TCP connection:

```go
tcpAddr := remote.(*net.TCPAddr)
```

Then:

```go
addr, ok := netip.AddrFromSlice(tcpAddr.IP)
if !ok {
	return
}

fmt.Println("client IP:", addr)
```

Now you can use all the `netip` functionality.

For example:

```go
private := addr.IsPrivate()
loopback := addr.IsLoopback()

fmt.Println("private:", private)
fmt.Println("loopback:", loopback)
```

---

# 6. Using `netip.Prefix` with TCP connections

This is a very practical use case.

Suppose your server only allows clients from:

```text
10.0.0.0/8
```

You can write:

```go
var allowedNetwork = netip.MustParsePrefix("10.0.0.0/8")
```

Then:

```go
func allowedClient(conn net.Conn) bool {
	tcpAddr, ok := conn.RemoteAddr().(*net.TCPAddr)
	if !ok {
		return false
	}

	addr, ok := netip.AddrFromSlice(tcpAddr.IP)
	if !ok {
		return false
	}

	return allowedNetwork.Contains(addr)
}
```

That's a very clean combination:

```text
TCP connection
      │
      ▼
net.TCPAddr
      │
      ▼
netip.Addr
      │
      ▼
netip.Prefix.Contains()
      │
      ▼
allowed / rejected
```

---

# 7. UDP

The same idea applies to UDP.

You can create an endpoint:

```go
addrPort := netip.MustParseAddrPort("127.0.0.1:9000")
```

Then:

```go
conn, err := net.ListenUDP(
	"udp",
	&net.UDPAddr{
		IP:   net.IP(addrPort.Addr().AsSlice()),
		Port: int(addrPort.Port()),
	},
)
if err != nil {
	panic(err)
}

defer conn.Close()
```

Here we're converting:

```text
netip.Addr
     ↓
AsSlice()
     ↓
net.IP
     ↓
net.UDPAddr
```

However, modern Go also provides newer APIs in `net` that work with `netip` more directly in several places, so you don't always need to manually construct `net.IP`.

---

# 8. `netip` and HTTP

This is where you'll encounter `netip` frequently in real Go applications.

Consider:

```go
func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Println(r.RemoteAddr)
}
```

You might see:

```text
192.168.1.20:54321
```

`r.RemoteAddr` is a string.

You can parse it with:

```go
addrPort, err := netip.ParseAddrPort(r.RemoteAddr)
if err != nil {
	http.Error(w, "invalid address", http.StatusBadRequest)
	return
}

ip := addrPort.Addr()

fmt.Println("client IP:", ip)
fmt.Println("client port:", addrPort.Port())
```

This is much nicer than manually splitting the string.

---

# 9. Important: `RemoteAddr` isn't necessarily the real client IP

This is an important HTTP networking concept.

Suppose:

```text
Client
   │
   ▼
Nginx / Load Balancer
   │
   ▼
Your Go server
```

Your Go server might see:

```text
10.0.0.5:43210
```

where `10.0.0.5` is the **proxy**, not the original client.

You may encounter headers such as:

```text
X-Forwarded-For
Forwarded
```

But you should **not blindly trust these headers**.

Only trust them when you know the request came through a proxy/load balancer that you control or otherwise trust.

---

# 10. `netip` and HTTP middleware

Once you understand that, you can build useful middleware.

For example, conceptually:

```go
var trustedNetwork = netip.MustParsePrefix("10.0.0.0/8")
```

You could determine whether the immediate peer is trusted:

```go
func isTrustedProxy(r *http.Request) bool {
	addrPort, err := netip.ParseAddrPort(r.RemoteAddr)
	if err != nil {
		return false
	}

	return trustedNetwork.Contains(addrPort.Addr())
}
```

Then only if the peer is trusted would you consider processing a forwarded-client-IP header.

This is much safer than:

```go
clientIP := r.Header.Get("X-Forwarded-For")
```

and assuming it's automatically trustworthy.

---

# 11. `netip` and DNS

Here's another important distinction.

DNS doesn't necessarily return an `netip.Addr` directly through the older APIs.

For example:

```go
ips, err := net.LookupIP("example.com")
if err != nil {
	panic(err)
}

for _, ip := range ips {
	fmt.Println(ip)
}
```

You get:

```text
[]net.IP
```

You can convert them:

```go
for _, ip := range ips {
	addr, ok := netip.AddrFromSlice(ip)
	if !ok {
		continue
	}

	fmt.Println(addr)
}
```

So:

```text
DNS
 │
 ▼
net.LookupIP()
 │
 ▼
net.IP
 │
 ▼
netip.Addr
```

---

# 12. Prefer `net.Resolver` when you need more control

For production networking code, you'll often use:

```go
resolver := net.Resolver{}
```

rather than relying only on the convenience functions.

For example:

```go
ctx := context.Background()

ips, err := resolver.LookupIP(ctx, "ip", "example.com")
if err != nil {
	panic(err)
}

for _, ip := range ips {
	addr, ok := netip.AddrFromSlice(ip)
	if ok {
		fmt.Println(addr)
	}
}
```

Again, you can convert the legacy `net.IP` values to `netip.Addr`.

---

# 13. The really useful modern API: `netip.AddrPort`

When writing networking code, try to think in terms of:

```text
Addr
Prefix
AddrPort
```

rather than manually manipulating strings.

Instead of:

```go
host := "192.168.1.10"
port := 8080

address := host + ":" + strconv.Itoa(port)
```

you can do:

```go
addr := netip.MustParseAddr(host)
endpoint := netip.AddrPortFrom(addr, uint16(port))

fmt.Println(endpoint)
```

And if you receive:

```text
[2001:db8::1]:8080
```

you don't need special IPv6 parsing logic:

```go
endpoint, err := netip.ParseAddrPort("[2001:db8::1]:8080")
```

---

# 14. `netip` and sockets: the bigger picture

A useful architecture to keep in your head is:

```text
                     netip
                       │
            ┌──────────┼──────────┐
            │          │          │
           Addr      Prefix    AddrPort
            │          │          │
            │          │          │
            ▼          ▼          ▼
           IP        subnet     endpoint
                                   │
                                   ▼
                                  net
                                   │
                 ┌─────────────────┼────────────────┐
                 │                 │                │
                TCP               UDP             DNS
                 │                 │                │
                 └─────────────────┼────────────────┘
                                   ▼
                                net/http
```

The important idea is that **`netip` is primarily about representing and reasoning about addresses**, while **`net` provides the actual networking operations**.

---

# 15. When should you use each?

### Use `netip.Addr` when:

You're dealing with an IP:

```go
netip.Addr
```

Examples:

```text
127.0.0.1
10.20.30.40
2001:db8::1
```

### Use `netip.Prefix` when:

You're dealing with a network:

```go
netip.Prefix
```

Examples:

```text
10.0.0.0/8
192.168.1.0/24
2001:db8::/32
```

### Use `netip.AddrPort` when:

You're dealing with an endpoint:

```go
netip.AddrPort
```

Examples:

```text
127.0.0.1:8080
[::1]:8080
10.0.0.5:443
```

### Use `net.Conn` when:

You're actually communicating:

```go
conn.Read(...)
conn.Write(...)
```

### Use `net.Listener` when:

You're accepting connections:

```go
listener.Accept()
```

### Use `net/http` when:

You're building HTTP applications:

```go
http.ListenAndServe(...)
http.Client
http.Server
```

---

# 16. One complete example

Let's put several pieces together.

Imagine a TCP server that:

1. Listens on `127.0.0.1:8080`
    
2. Accepts connections
    
3. Extracts the client's IP
    
4. Converts it to `netip.Addr`
    
5. Checks whether it's loopback
    
6. Sends a response
    

```go
package main

import (
	"fmt"
	"net"
	"net/netip"
)

func main() {
	listenAddr := netip.MustParseAddrPort("127.0.0.1:8080")

	listener, err := net.Listen("tcp", listenAddr.String())
	if err != nil {
		panic(err)
	}
	defer listener.Close()

	fmt.Println("listening on", listenAddr)

	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("accept:", err)
			continue
		}

		go handle(conn)
	}
}

func handle(conn net.Conn) {
	defer conn.Close()

	tcpAddr, ok := conn.RemoteAddr().(*net.TCPAddr)
	if !ok {
		return
	}

	clientIP, ok := netip.AddrFromSlice(tcpAddr.IP)
	if !ok {
		return
	}

	fmt.Println("client:", clientIP)
	fmt.Println("loopback:", clientIP.IsLoopback())

	_, _ = conn.Write([]byte("hello\n"))
}
```

Notice how the code naturally crosses the boundary:

```text
netip.AddrPort
      ↓
     net
      ↓
TCP connection
      ↓
net.TCPAddr
      ↓
netip.Addr
      ↓
IsLoopback()
```

That's the relationship I'd focus on when learning Go networking.

**One especially useful next topic is `netip.Prefix`:** CIDR notation, subnet masks, `Bits()`, `Masked()`, `Contains()`, IPv4-mapped IPv6 addresses, and how to implement an IP allowlist/denylist correctly.