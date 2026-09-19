
### Why is IP address broken into Network and Host  part

An IP address is divided into a **network part** and a **host part** so that devices can be identified efficiently and data can be routed correctly across networks.

Here's why this separation exists:

1. **Identifies the network**

   * The **network part** tells routers which network a device belongs to.
   * Routers use this information to forward packets toward the correct destination network without needing to know every individual device.

2. **Identifies the device (host)**

   * The **host part** uniquely identifies a specific device within that network.
   * Once the packet reaches the correct network, the local network uses the host part to deliver it to the intended device.

### Example

Consider the IPv4 address:

**192.168.1.25/24**

* Network prefix (/24): **192.168.1**
* Host part: **25**

Here:

* **192.168.1.0** represents the network.
* **25** identifies a particular device on that network.


### Visual representation

```
IPv4 Address: 192.168.1.25/24

+----------------------+------------+
|    Network Part      | Host Part  |
+----------------------+------------+
|      192.168.1       |     25     |
+----------------------+------------+
```

The exact boundary between the network and host portions is determined by the **subnet mask** (or **prefix length**, such as `/24`). Different subnet masks allocate different numbers of bits to the network and host portions depending on the size and needs of the network.



## IPv6 vs IPv4
### Why is the host part in IPv6 called the `interface id` 

That's a good observation. The terminology reflects a design difference between IPv4 and IPv6.

### IPv4: Host portion

In IPv4, the address is usually described as:

```
| Network | Host |
```

For example:

```
192.168.1.42/24

Network: 192.168.1.0
Host:    42
```

The "host" part simply identifies a device within the subnet.

### IPv6: Interface Identifier (IID)

IPv6 typically describes the address as:

```
| Network Prefix | Interface Identifier (IID) |
```

For example:

```
2001:db8:abcd:1234:021c:7eff:fe12:3456/64
```

* Network Prefix:

  ```
  2001:db8:abcd:1234
  ```
* Interface Identifier (last 64 bits):

  ```
  021c:7eff:fe12:3456
  ```

The term **Interface Identifier** is preferred because an IPv6 address identifies a **network interface**, not necessarily an entire host.

### Why "interface" instead of "host"?

A single device (host) can have multiple network interfaces, and each interface can have one or more IPv6 addresses.

For example:

The laptop is one host, but it has four interfaces
```
Laptop
├── Wi-Fi interface
├── Ethernet interface
├── VPN interface
└── Docker interface
```

```
Laptop
├── Wi-Fi
│   └── 2001:db8:1::10/64
├── Ethernet
│   └── 2001:db8:2::20/64
└── VPN
    └── 2001:db8:100::5/64
```

Each interface can have one or more ipv6 addresses. Suppose your Wi-Fi interface is `wlan0`.

```
wlan0

fe80::21a:2bff:fe3c:4d5e
↑ Link-local address

2001:db8:abcd:1::25
↑ Global unicast address

2001:db8:abcd:1::100
↑ Temporary/privacy address

fd12:3456:789a::25
↑ Unique local address
```

These all serve different purposes:

- **Link-local** (`fe80::/10`) → communication only on the local network.
- **Global unicast** → reachable on the Internet.
- **Unique local** (`fc00::/7` or `fd00::/8`) → private IPv6 networks.
- **Temporary/privacy** → changes periodically to reduce tracking.

Having several IPv6 addresses on one interface is completely normal.

Each interface has its own Interface Identifier within its subnet.

For example:
```
2001:db8:1234:5678:1111:2222:3333:4444
```
is the network.  A device could have:
```
2001:db8:1234:5678:1111:2222:3333:4444
```
Another device on the same network could have:
```
2001:db8:1234:5678:aaaa:bbbb:cccc:dddd
```


This distinction is also useful because IPv6 addresses aren't assigned only to hosts. Routers, tunnel endpoints, virtual interfaces, and even a single physical interface may have multiple IPv6 addresses (such as link-local, global unicast, temporary, and unique local addresses).

### Is the IID always 64 bits?

For the vast majority of LANs, **yes**. A `/64` prefix leaves the remaining 64 bits as the Interface Identifier:

```
|<--------- 64 bits --------->|<--------- 64 bits --------->|
+-----------------------------+-----------------------------+
|       Network Prefix        |     Interface Identifier    |
+-----------------------------+-----------------------------+
```

However, not all IPv6 prefixes are `/64`:

* `/128` → no IID remains (a single address).
* `/127` → only 1 bit is available for the interface identifier, commonly used on point-to-point links.
* `/126` → 2 bits for the interface identifier.
* `/120` → 8 bits for the interface identifier.

So "Interface Identifier" refers to **whatever bits remain after the network prefix**, even though the classic 64-bit IID is by far the most common case.

### Historical note

Originally, many IPv6 Interface Identifiers were automatically generated from a network card's MAC address using a method called **Modified EUI-64**. Today, operating systems more commonly generate IIDs randomly or pseudorandomly (using privacy extensions or stable opaque identifiers) to reduce device tracking. This is another reason the term "Interface Identifier" is more appropriate than "host ID"—it identifies an interface on a specific network, not the device as a whole.

### FYI

IPv4 also supports multiple addresses on a single interface.
```
eth0

192.168.1.10/24
192.168.1.20/24
10.0.0.5/8
```
The operating system treats all of these as belonging to the same interface.

This is sometimes called **IP aliasing** (although modern systems simply treat them as multiple assigned addresses rather than true "aliases").

-------------

# CDN and how is the nearest edge chosen?

A **Content Delivery Network (CDN)** is a globally distributed network of servers that stores copies (or caches) of website content closer to users. The main goal is to make websites load faster, reduce the load on the origin server, and improve availability.

Here's how it works.

### Without a CDN

Imagine your website is hosted on a single server in New York.

```
User (India)
      |
      |  Request
      |
Origin Server (New York)
      |
      |  Response
      |
User receives page
```

A user in India has to wait for data to travel halfway around the world, which increases latency.

---

### With a CDN

Now suppose you use a CDN with edge servers around the world.

```
                    Origin Server
                    (New York)
                          |
          -------------------------------
         |             |                |
     Edge Server   Edge Server    Edge Server
      (Mumbai)      (London)       (Tokyo)
         |              |              |
      Indian User    UK User      Japan User
```

Instead of contacting the origin server every time, users connect to the nearest edge server.

---

## Step-by-step request flow

### 1. User requests a webpage

A user visits:

```
https://example.com/logo.png
```

The browser performs a DNS lookup.

Instead of returning the IP of the origin server, DNS returns the IP address of the nearest CDN edge server.

---

### 2. Request reaches the nearest edge

For a user in Bangalore, the request might go to the Mumbai edge.

```
Browser
   |
   |
Mumbai Edge Server
```

---

### 3. Check the cache

The edge server checks:

```
Is logo.png already cached?
```

### Case A: Cache Hit ✅

If it exists:

```
Browser
   |
   |
Mumbai Edge
   |
Cached logo.png
```

The edge immediately returns the file.

The origin server is never contacted.

This is very fast.

---

### Case B: Cache Miss ❌

If the file isn't cached:

```
Browser
   |
Mumbai Edge
   |
Origin Server
```

The edge requests the file from the origin server.

The origin responds:

```
Origin
   |
logo.png
   |
Mumbai Edge
```

The edge:

* stores the file in its cache
* sends it to the user

Future users in that region get the cached copy.

---

## What kinds of files are cached?

Typically:

* Images
* CSS
* JavaScript
* Fonts
* Videos
* PDFs
* Downloadable files

Many CDNs can also cache:

* HTML pages
* API responses
* GraphQL responses
* JSON
* Server-rendered pages

---

## Cache expiration

Cached content shouldn't stay forever.

The origin server sends headers like:

```
Cache-Control: max-age=3600
```

Meaning:

```
Cache for 1 hour
```

After one hour:

```
Edge Server
      |
Cache expired
      |
Fetch fresh version
```

---

## Example

Suppose your homepage has:

```
HTML
10 images
2 CSS files
5 JS files
```

Without a CDN:

Every visitor downloads everything from your origin.

With a CDN:

```
First visitor

Origin ---> CDN ---> User

Second visitor

CDN ---> User

Third visitor

CDN ---> User
```

Only the first request (or the first after cache expiry) reaches the origin.

---

## Dynamic content

Not everything can be cached.

Examples:

* Bank account balance
* Shopping cart
* User profile
* Live notifications

These usually go to the origin server because they're personalized.

Some CDNs use techniques like edge computing to cache parts of a page while fetching only the personalized sections from the origin.

---

## How the "nearest server" is chosen

CDNs use routing techniques such as:

* **DNS-based routing**: DNS returns the IP of a nearby edge server.
* **Anycast routing**: Multiple edge servers advertise the same IP address, and internet routing naturally sends the request to the closest or best-performing one.

---

## Why CDNs improve performance

### 1. Lower latency

Instead of:

```
India → USA → India
```

the request becomes:

```
India → Mumbai → India
```

Less distance means lower latency.

---

### 2. Less origin traffic

Instead of:

```
1,000,000 users
        |
Origin Server
```

you get:

```
1,000,000 users
        |
    CDN Edge Servers
        |
Only occasional requests
        |
Origin Server
```

This reduces bandwidth usage and server load.

---

### 3. Better scalability

If a viral post suddenly attracts millions of visitors, the CDN can serve most requests from its edge caches instead of overwhelming the origin server.

---

### 4. Improved reliability

If one edge server fails, traffic can be routed to another nearby edge, helping keep content available.

---

# DNS Based routing and Nearest edge selection 

DNS-based routing is one of the clever tricks CDNs use to direct users to the "best" edge server **before any HTTP request is even sent**.

Let's walk through it from the moment you type a URL.

---

## Step 1: User enters a URL

Suppose you type

```
https://example.com
```

The browser first needs the IP address.

It asks:

```
What's the IP address of example.com?
```

---

## Step 2: Browser asks its DNS resolver

The browser doesn't directly ask the CDN.

Instead it asks a DNS resolver, usually provided by:

* Google DNS (8.8.8.8)
* Cloudflare DNS (1.1.1.1)
* ISP DNS
* Corporate DNS

```
Browser
    |
    |
DNS Resolver
```

---

## Step 3: Resolver queries authoritative DNS

The resolver asks the authoritative DNS server for example.com.

```
Browser
   |
DNS Resolver
   |
Authoritative DNS
```

If the website uses a CDN, the authoritative DNS server usually belongs to the CDN.

For example:

```
example.com

↓

Authoritative DNS

↓

CDN DNS
```

---

## Step 4: CDN determines the user's location

Here's the interesting part.

The CDN doesn't usually know **your exact IP address**.

Instead, it sees the IP address of the **DNS resolver** making the query.


Example:

```
You (Bangalore)

↓

Google DNS (Bangalore)

↓

CDN DNS
```

The The Authoritative server that belongs to the CDN i.e  CDN looks up the resolver's location.

Suppose it determines:

```
Resolver location:
Bangalore
```

---

## Step 5: CDN selects the nearest edge

The CDN has many edge servers.

```
Mumbai
Delhi
Singapore
Tokyo
London
New York
```

It estimates:

```
Resolver location = Bangalore

Closest edge = Mumbai
```

---

## Step 6: DNS returns Mumbai's IP

Instead of returning the origin server:

```
Origin

52.100.10.1
```

it returns

```
Mumbai Edge

203.0.113.5
```

The resolver sends this back to your browser.

```
Browser

↓

203.0.113.5
```

---

## Step 7: Browser connects directly

Now the browser opens a TCP/TLS connection directly to

```
203.0.113.5
```

which is the Mumbai CDN edge.

The HTTP request goes there.

```
Browser
      |
Mumbai Edge
```

No request goes to the origin unless needed.

---

## Complete flow

```
Browser
    |
    | DNS Query
    |
DNS Resolver
    |
    | Query
    |
CDN DNS
    |
    | "Nearest edge is Mumbai"
    |
Returns Mumbai IP
    |
DNS Resolver
    |
Returns IP
    |
Browser
    |
HTTP Request
    |
Mumbai Edge
```

Notice that DNS is only used to answer **"Where should I connect?"**. After that, the browser communicates directly with the chosen edge server.

---

## How does the CDN know which server is "nearest"?

It's not just geographic distance.

CDNs continuously measure:

* network latency
* packet loss
* congestion
* server load
* edge server health
* ISP connectivity

Example:

```
User: Bangalore

Mumbai latency:
18 ms

Singapore latency:
25 ms

Delhi latency:
42 ms

Tokyo latency:
120 ms
```

The CDN chooses Mumbai because it offers the best overall path.

---

## DNS responses are cached

DNS isn't queried for every request.

The response includes a **TTL (Time To Live)**.

Example:

```
example.com

TTL = 60 seconds
```

That means:

```
Next 60 seconds

↓

Reuse Mumbai IP

↓

No DNS query
```

After the TTL expires, the browser or resolver asks again, giving the CDN a chance to direct traffic differently if conditions have changed.

---

## What if the Mumbai server goes down?

The CDN can simply return a different IP on the next DNS lookup.

```
Before

example.com

↓

Mumbai IP
```

Later:

```
example.com

↓

Singapore IP
```

Because DNS entries have relatively short TTLs (often 30–300 seconds for CDNs), users can be redirected fairly quickly.

---

## A limitation of DNS-based routing

Since the CDN often sees the **DNS resolver's** location rather than the end user's, it can sometimes make a suboptimal choice.

For example:

```
User: Bangalore

Uses DNS Resolver:
New York
```

The CDN might incorrectly assume the user is in New York and return a New York edge server.

To improve accuracy, many DNS resolvers support the **EDNS Client Subnet (ECS)** extension, which includes part of the user's IP prefix in the DNS query. This lets the CDN estimate the user's location more accurately without exposing the full IP address.

---

# DNS-based routing vs. Anycast routing

| DNS-based routing                                     | Anycast routing                                           |
| ----------------------------------------------------- | --------------------------------------------------------- |
| Chooses the server during DNS resolution.             | Chooses the server during network routing.                |
| Returns different IP addresses for different users.   | Many edge servers share the same IP address.              |
| Decisions are made by the CDN's DNS service.          | Decisions are made by Internet routing (BGP).             |
| Can change the returned IP after the DNS TTL expires. | Traffic can shift automatically as network routes change. |

Many large CDNs actually **combine both techniques**. DNS first directs users to an appropriate region or service, and then Anycast routing ensures traffic reaches the best available edge server within that region. This layered approach provides both flexibility and resilience.
