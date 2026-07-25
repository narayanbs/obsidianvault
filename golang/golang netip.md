
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

## Key Takeaways

1. **Default Choice:** Use `net/netip` for any new Go project handling IP routing, rate-limiting, firewalling, or networking calculations.
    
2. **Performance:** Offers **0 heap allocations** for address manipulation, resulting in lower GC latency in high-throughput network applications.
    
3. **Map Keys:** Unlocks direct use of IPs and subnets in maps, set structures, and hash sets natively.