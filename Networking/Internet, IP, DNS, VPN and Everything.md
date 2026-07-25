
# How are IP addresses handed over 
## Internet Hierarchy

Let's see how the Internet is governed, and it's a fascinating hierarchy. There isn't a single "global Internet registry" that hands out every IP address directly. Instead, there's a well-defined chain of delegation.

Here's the hierarchy:

```text
IANA (Global)
        │
        ▼
Regional Internet Registries (RIRs)
        │
        ▼
ISPs / Large Organizations
        │
        ▼
Customers
```

## 1. IANA – The global coordinator

At the top is the **Internet Assigned Numbers Authority (IANA)**.

IANA is responsible for:

* The global pool of IPv4 and IPv6 addresses.
* The DNS root zone (the list of top-level domains like `.com`, `.org`, `.in`).
* Autonomous System Numbers (ASNs), which are used for Internet routing.

IANA does **not** normally assign IP addresses directly to ISPs or individuals. Instead, it allocates large address blocks to the Regional Internet Registries.

Today, IANA's functions are operated by the **ICANN** under a community-driven governance model.

---

## 2. Regional Internet Registries (RIRs)

The world is divided into five regions, each served by an RIR:

| Region                                           | Registry |
| ------------------------------------------------ | -------- |
| North America                                    | ARIN     |
| Europe, Middle East, parts of Central Asia       | RIPE NCC |
| Asia-Pacific (including India, Australia, Japan) | APNIC    |
| Latin America                                    | LACNIC   |
| Africa                                           | AFRINIC  |

Since you're in India, Indian ISPs typically obtain IP address allocations from **APNIC**.

---

## 3. ISP obtains an allocation

Suppose a new company wants to become an ISP in India.

They would typically:

* Become a member of APNIC.
* Demonstrate that they have a legitimate need for IP addresses.
* Request IP address space and usually an Autonomous System Number (ASN).

For IPv6, APNIC can allocate very large address blocks because the address space is enormous.

For IPv4, things are different because the free global pool has been exhausted. New ISPs often:

* receive only small allocations (if available),
* buy address blocks from other organizations,
* or rely heavily on Carrier-Grade NAT.

---

## 4. What is an ASN?

An **Autonomous System Number (ASN)** identifies a network on the Internet.

Think of it as a unique ID for an organization that operates a network.

For example:

```text
Home User
      │
ISP A
ASN 12345
      │
──────── Internet ────────
      │
Google
ASN 15169
      │
Amazon
ASN 16509
```

Routers on the Internet exchange routing information using the **Border Gateway Protocol (BGP)**, announcing which IP address ranges each ASN can reach.

For example, an ISP might advertise:

```text
ASN 12345

Owns:

203.0.113.0/20
```

Other networks then know that traffic destined for those IP addresses should be sent toward ASN 12345.


### Border Gateway Protocol

Think of the internet not as a single, massive highway, but as a collection of thousands of smaller, independent networks (like individual neighborhoods, universities, or massive tech companies). These independent networks are called **Autonomous Systems (AS)**.

If you want to send data from your computer to a server on the other side of the world, your data needs to hop across several of these Autonomous Systems. **Border Gateway Protocol (BGP)** is the postal service of the internet that decides the best, most efficient path for your data to take.

Here is a breakdown of how it works, using a real-world analogy.

---

## The Analogy: The International Postal Network

Imagine you are in **New York (AS 1)** and want to send a letter to a friend in **Tokyo (AS 4)**.

There is no direct mail truck from your house to theirs. Instead, the post office in New York looks at its global routing map. It sees a few options:

* **Path A:** New York $\rightarrow$ London (AS 2) $\rightarrow$ Tokyo (AS 4) *(2 hops)*
* **Path B:** New York $\rightarrow$ Los Angeles (AS 3) $\rightarrow$ Sydney (AS 5) $\rightarrow$ Tokyo (AS 4) *(3 hops)*

BGP acts like the mastermind postmaster. It evaluates the paths and says, *"Path A is shorter and less congested right now. Let's send it through London."*

---

## Technical Example: How BGP Works in Action

Let's translate this into actual internet routing. Imagine three internet service providers (ISPs) and one major website:

* **AS 100 (Your Home ISP):** Where you are sitting.
* **AS 200 (Transit Provider A):** A massive network spanning the Atlantic.
* **AS 300 (Transit Provider B):** A massive network spanning the Pacific.
* **AS 400 (Netflix Servers):** Where the data needs to go.

### 1. Advertising the Routes (The "Hey, I'm here!" phase)

Netflix (AS 400) wants the world to know how to reach its servers. Its BGP routers send out an **advertisement** to its neighbors (AS 200 and AS 300):

> *"Hey, if anyone wants to reach Netflix, come to me (AS 400)."*

### 2. Passing the Message Along

Now, AS 200 and AS 300 update their own routing tables and pass the message to your home ISP (AS 100). They attach their own names to the message:

* **AS 200 tells AS 100:** *"I can get you to Netflix. The path is: AS 200 $\rightarrow$ AS 400."*
* **AS 300 tells AS 100:** *"I can also get you to Netflix. The path is: AS 300 $\rightarrow$ AS 400."*

### 3. Making the Decision (Path Vector Routing)

Your home ISP (AS 100) now has two choices to put in its BGP routing table. BGP uses a set of rules (metrics) to decide the best path. The most fundamental rule is the **AS-Path length** (the fewest number of network hops).

If AS 200 suddenly encounters a major fiber-optic cable cut, it might update its path to look like: `AS 200 -> AS 500 -> AS 400` (3 hops).

AS 100's BGP router will look at both options:

| Destination | Available Paths | Decision |
| --- | --- | --- |
| **Netflix** | `AS 200 -> AS 500 -> AS 400` (3 hops) | Rejected (Too long) |
| **Netflix** | `AS 300 -> AS 400` (2 hops) | **Selected (Best Path)** |

Your ISP chooses the **AS 300** route, and your video starts streaming seamlessly.

---

## Why BGP is Critical: Handling Failures

The internet is dynamic. If a construction crew accidentally digs up an internet cable in AS 300, that network will immediately send out a **BGP withdrawal notice** saying, *"Hey, I can no longer reach Netflix!"*

Within seconds, your home ISP’s BGP router will dynamically switch over to the backup route (`AS 200 -> AS 500 -> AS 400`). You might notice a tiny stutter in your video, but the internet keeps working because BGP automatically rerouted the traffic around the failure.

---

## 5. Can anyone become an ISP?

Yes, although there are practical and regulatory requirements.

Typically, a company would need:

* Telecommunications licenses required by its country.
* Connectivity to one or more upstream Internet providers or Internet Exchange Points (IXPs).
* Routers capable of running BGP.
* IP address allocations.
* An ASN.

Small ISPs often purchase Internet transit from larger ISPs.

A simplified picture looks like this:

```text
                 Tier 1 ISP
                      │
          ┌───────────┴───────────┐
          │                       │
      Tier 2 ISP             Tier 2 ISP
          │                       │
      Small ISP             Small ISP
          │
      Home Customers
```

---

## 6. Who owns the Internet?

This is one of the most interesting aspects of the Internet: **no single organization owns it.**

Different organizations are responsible for different parts:

* **ICANN/IANA** coordinate the global IP address space, DNS root zone, and protocol parameter registries.
* **Regional Internet Registries (RIRs)** allocate IP addresses and ASNs within their regions.
* **Domain registries** manage specific top-level domains (for example, the `.com` registry or the `.in` registry).
* **ISPs** build and operate networks that connect users.
* **Internet Exchange Points (IXPs)** allow networks to interconnect and exchange traffic efficiently.
* **Standards bodies** such as the Internet Engineering Task Force develop the technical protocols (like TCP, IP, HTTP, DNS, and BGP) that make the Internet interoperable.

---

### A useful analogy

Think of the Internet like a postal system:

* **IANA** decides how postal code regions are divided globally.
* **RIRs** assign blocks of postal codes to countries or large delivery organizations.
* **ISPs** are like local postal services that receive a range of addresses to serve.
* **Your home** gets one address within that range.
* **DNS** is like the phone book that maps a name ("John's Bakery") to a street address.

Each organization has a clearly defined role, and together they make it possible for billions of devices to communicate over a single, interoperable global Internet.


## So the ISP are handed a set of IPs,  how do they allocate them to customers

There are **two common models**, and both are used today.

## Model 1: ISP assigns you a unique public IP

This is the traditional model.

The ISP owns (or leases) a block of public IP addresses from a Regional Internet Registry (RIR). For example:

```text
ISP owns:

203.0.113.0/24
```

This means the ISP controls 256 IPv4 addresses (actually 254 usable host addresses in a typical subnet).

When you connect to the Internet, the ISP assigns one of these to your router:

```text
You       → 203.0.113.25
Neighbor  → 203.0.113.26
Friend    → 203.0.113.27
```

Every customer gets a unique public IP (at least while it is assigned to them).

Your home router then performs NAT for the devices inside your house:

```text
Laptop      192.168.1.10
Phone       192.168.1.11
TV          192.168.1.12
        │
Home Router (NAT)
        │
203.0.113.25
        │
Internet
```

This is the NAT most people are familiar with.

---

## Model 2: ISP uses Carrier-Grade NAT (CGNAT)

IPv4 addresses are scarce. Many ISPs no longer have enough public addresses to give every customer a unique one.

Instead, they use **Carrier-Grade NAT (CGNAT)**.

Your router receives an IP that is **not globally routable**, for example:

```text
100.64.10.25
```

Notice this is in the special range `100.64.0.0/10`, reserved specifically for CGNAT.

Then the ISP has a large NAT gateway:

```text
Your PC
192.168.1.10
      │
Home Router (NAT)
100.64.10.25
      │
ISP CGNAT
203.0.113.80
      │
Internet
```

Now hundreds or even thousands of customers may share the same public IP:

```text
Customer A
100.64.1.10
        │

Customer B
100.64.5.20
        │

Customer C
100.64.9.30
        │

ISP NAT
        │
203.0.113.80
```

The ISP's NAT device keeps track of each connection using source ports, just like your home router does.

## How do we check if our ISP uses CGNAT 

1. Find your Public IP: Open a browser and go to a site like whatsmyip.org or ipchicken.com. Note the IP address shown.
2. Find your Router's WAN IP:  Log into your router’s admin panel (usually by typing 192.168.0.1 or 192.168.1.1 into your browser). Look for the Status, WAN, or Internet page and find the IPv4 Address listed there.
3. Compare them:
    If the two IP addresses match: You have a public IP address (No CGNAT).
    If the two IP addresses are different: You are likely behind CGNAT.

 If  the WAN ip falls anywhere within the following range, your ISP is definitely using CGNAT:
    `100.64.0.0 to 100.127.255.255`
    

## I have noticed that my home router WAN IP is in the 100.x.x.x range, so my ISP uses CGNAT, so what is the 192.168.1.1 ip that my home router uses?

You're mixing two different interfaces on the same router. Your home router typically has **at least two IP addresses**:

1. **LAN IP** (inside your home network)
2. **WAN IP** (toward your ISP)

Think of the router as standing between two different networks.

```
Internet
    |
Public IP (ISP's CGNAT router)
    |
100.x.x.x  <-- Your router's WAN interface
+-----------------------+
|     Home Router       |
|                       |
| WAN: 100.72.15.34     |
| LAN: 192.168.1.1      |
+-----------------------+
    |
-------------------------------
|             |               |
192.168.1.10 192.168.1.20 192.168.1.30
Laptop        Phone        TV
```

### What is 192.168.1.1?

This is the **LAN IP address of your router**.

* Your devices use it as the **default gateway**.
* It's the address you use to log into your router's admin page.
* It exists only inside your home network.

When your laptop sends a packet to the internet:

```
Laptop (192.168.1.10)
        |
        v
Gateway (192.168.1.1)
```

The router receives the packet on its LAN interface.

---

### What is the 100.x.x.x address?

This is the **WAN IP address of the same router**.

It is assigned by your ISP to the router's **WAN interface**.

For example:

```
Router LAN IP: 192.168.1.1
Router WAN IP: 100.83.42.117
```

Both belong to the same physical router, but on different network interfaces.

---

### Why is 100.x.x.x not public?

The range **100.64.0.0/10** (100.64.0.0–100.127.255.255) is reserved specifically for Carrier-Grade NAT (CGNAT).

So if your router receives:

```
WAN IP = 100.83.42.117
```

the path is likely:

```
Laptop
192.168.1.10
      |
Router
LAN: 192.168.1.1
WAN: 100.83.42.117
      |
ISP CGNAT Router
Public IP: 203.45.67.89
      |
Internet
```

There are actually **two layers of NAT**:

1. **Your router**

   * `192.168.1.x` → `100.83.42.117`

2. **ISP's CGNAT**

   * `100.83.42.117` → `203.45.67.89`

---

### So which IP belongs to the home router?

**Both do.**

The router has:

* **LAN IP:** `192.168.1.1`

  * Used by devices inside your home.
  * Default gateway.

* **WAN IP:** `100.x.x.x`

  * Used by the ISP's network.
  * Assigned to the router's WAN port.

They're simply addresses on two different interfaces of the same device.

---

### An analogy

Imagine your house:

* **Front door address:** `100.x.x.x` (known to the ISP's network)
* **Room number inside the house:** `192.168.1.1` (known only within your home)

They're both associated with the same house, just from different perspectives.

---

### How to confirm you're behind CGNAT

Compare:

* **Router WAN IP:** `100.x.x.x`
* **"What is my IP" website:** e.g., `203.45.67.89`

If these are different, and your WAN IP is in `100.64.0.0/10`, then you're almost certainly behind CGNAT.

If they are the **same**, then you have a public IP directly assigned to your router (unless another unusual network setup is involved).



---

## Why is CGNAT a problem?

Because **incoming connections can't easily reach your home**.

Suppose someone wants to visit your website.

With a unique public IP:

```text
Internet
     │
203.0.113.25
     │
Your Router
     │
Web Server
```

The router can forward port 443 to your web server.

With CGNAT:

```text
Internet
     │
203.0.113.80
     │
ISP NAT
     │
100.64.10.25
     │
Your Router
     │
Web Server
```

The incoming connection stops at the ISP's NAT device. Since you don't control that NAT, you typically can't configure port forwarding there.

So  If your activities are:
```
Browsing websites
Watching YouTube or Netflix
Using WhatsApp
Video calls (Zoom, Teams, Meet)
Online shopping
Using cloud storage
Sending emails
```
...then your device always initiates the connection. CGNAT keeps track of that connection and routes the replies back correctly.

It becomes a problem when you want someone else to initiate a connection to you.

For example:

❌ Hosting a website at home
```
Internet
    │
https://john.com
    │
Your Home Server
```
❌ Running your own mail server
```
Internet
    │
SMTP
    │
Your Mail Server
```
❌ Hosting a game server
❌ Running an SSH server

This is why people behind CGNAT often cannot host services directly from home unless they use workarounds like a VPN with port forwarding, a reverse tunnel, or obtain a public IP from their ISP.

---
## How can you tell if you're behind CGNAT?

Compare:

* **Your router's WAN IP address** (shown in the router's admin page).
* **Your public IP** (shown by websites like "What is my IP?").

If your router says:

```text
WAN IP: 100.64.25.18
```

but a website reports:

```text
Public IP: 203.0.113.80
```

then you're almost certainly behind CGNAT.

If both show the same public IP, for example:

```text
WAN IP: 203.0.113.25
Public IP: 203.0.113.25
```

then you have a public IP assigned directly to your router.

### Where do ISPs get these public IP addresses?

ISPs receive allocations from Regional Internet Registries (RIRs), such as:

* ARIN (North America)
* RIPE NCC
* APNIC (Asia-Pacific)
* LACNIC
* AFRINIC

These organizations allocate blocks of IP addresses to ISPs, and the ISPs then assign or share those addresses among their customers.

So to summarize

```
Private devices
      │
Home NAT
      │
(Unique Public IP)  ← Traditional model
      │
Internet
```

or

```
Private devices
      │
Home NAT
      │
CGNAT address (100.64.0.0/10)
      │
ISP NAT
      │
Shared Public IP
      │
Internet
```

The second model—**NAT inside your home plus another NAT inside your ISP**—is exactly what Carrier-Grade NAT is. It's often described as **"double NAT."**

## so why CGNAT?

The primary reason is simple: **there aren't enough IPv4 addresses for every Internet connection.**

CGNAT is essentially a way for ISPs to let many customers share a small pool of public IPv4 addresses.

### Why is this necessary?

IPv4 uses **32-bit addresses**, giving about:

```text
2^32 = 4,294,967,296 addresses
```

About 4.3 billion sounds like a lot, but:

* Some ranges are reserved (private networks, multicast, loopback, etc.).
* Many addresses were allocated years ago.
* Today there are billions of phones, laptops, TVs, IoT devices, and servers.

By around **2011**, IANA had allocated the last blocks of free IPv4 addresses to the Regional Internet Registries. Since then, IPv4 addresses have become a scarce resource.

---

## What would happen without CGNAT?

Imagine an ISP has:

* 5 million customers
* only 500,000 public IPv4 addresses

Without CGNAT:

```text
Customers:        5,000,000
Public IPs:         500,000
```

Only 10% of customers could be online at once.

With CGNAT:

```text
500,000 public IPs
        │
CGNAT
        │
5,000,000 customers
```

Every customer can access the Internet because the ISP shares public IPs among many users.

---

## How can many people share one IP?

Because most Internet traffic is **outbound**.

Suppose three customers all browse websites:

```text
Alice → google.com
Bob   → youtube.com
Carol → github.com
```

Even if they all share the same public IP:

```text
203.0.113.80
```

the NAT device distinguishes them by **port numbers**.

For example:

| Customer | Internal Port | Public Port |
| -------- | ------------: | ----------: |
| Alice    |         50123 |       40001 |
| Bob      |         50123 |       40002 |
| Carol    |         50123 |       40003 |

To the outside world, all three connections come from `203.0.113.80`, but with different source ports. Replies are routed back to the correct customer.

---

## Why not just use IPv6?

That's the long-term solution.

IPv6 has **128-bit addresses**, providing an unimaginably large address space. Every home, and even every device, can have globally unique addresses.

However, adoption has taken time because:

* Existing equipment needed upgrades.
* Many older systems were built only for IPv4.
* The entire Internet couldn't switch overnight.

Today, many ISPs run **dual stack**:

```text
Customer
   │
IPv4 (CGNAT)
IPv6 (native)
```

Customers use IPv6 whenever possible, and IPv4 for services that don't yet support IPv6.

---

## Are there downsides to CGNAT?

Yes.

* You usually can't host servers from home.
* Port forwarding typically isn't possible.
* Some peer-to-peer applications and online games may have connectivity issues.
* If many users share one public IP, abuse by one customer can affect the reputation of that IP (for example, triggering rate limits or CAPTCHAs for others).
* ISPs must keep detailed logs mapping public IPs and ports to customers for troubleshooting and, where required by law, legal compliance.

---

## Why do businesses often get public IPs?

Businesses frequently need to:

* host websites,
* run VPN servers,
* provide email servers,
* allow remote access to office systems.

Those use cases require inbound connections, so business Internet plans often include one or more dedicated public IP addresses.

---

### An analogy

Imagine a large office building.

Without CGNAT:

* Every employee has their own mailing address.

With CGNAT:

* The whole building has one street address:

  ```
  123 Main Street
  ```
* Each employee is identified internally by their office number.

When someone inside the building sends mail, the mailroom knows which office it came from and can route replies correctly.

But if someone on the outside sends a letter addressed only to:

```
123 Main Street
```

with no office number, the mailroom has no idea which employee should receive it.

CGNAT works in much the same way: it efficiently supports **outbound communication** for many customers sharing one public IP, but it doesn't inherently know where to send **new inbound connections** unless there's additional configuration or infrastructure.

-------------------

# New websites and DNS

## Say i create a new website `john.com` and i host it by getting a static I/p from my ISP or  i use a provider, how is the address mapped to DNS

 **Who is responsible for the DNS records for `john.com`?**

### Step 1: You register `john.com`

Suppose you buy `john.com` from a registrar like GoDaddy, Namecheap, or Cloudflare Registrar.

At this point:

* You own the domain `john.com`.
* You choose a DNS provider (which could be the registrar itself, Cloudflare DNS, AWS Route 53, etc.).

Your DNS provider hosts the DNS zone for `john.com`.

For example, the DNS zone might contain:

```
john.com.        A       203.0.113.10
john.com.        A       203.0.113.11
john.com.        AAAA    2001:db8::10

www.john.com.    A       203.0.113.20

mail.john.com.   A       203.0.113.30

john.com.        MX      mail.john.com.
```

These records are stored **only on your authoritative DNS servers**.

---

## Step 2: What gets stored at the TLD (.com) servers?

When you register the domain, your registrar tells the `.com` registry:

> "The authoritative name servers for john.com are:"

For example:

```
john.com

NS  ns1.cloudflare.com.
NS  ns2.cloudflare.com.
```

That's all.

The `.com` registry **does not know your website's IP address.**

It only knows where the authoritative DNS servers are.

---

## Step 3: What do the root servers store?

The root servers know only about **Top-Level Domains (TLDs).**

For example:

```
.
├── com
├── org
├── net
├── edu
├── gov
```

The root zone contains records like:

```
com.

NS a.gtld-servers.net.
NS b.gtld-servers.net.
NS c.gtld-servers.net.
...
```

It does **not** contain:

```
john.com
google.com
amazon.com
```

Those belong to the `.com` servers.

---

## Step 4: Complete lookup

Suppose your browser wants the IP for `john.com`.

### Query 1

Ask a root server:

```
What is john.com?
```

Root server replies:

> I don't know.
>
> Ask the .com servers.

```
NS a.gtld-servers.net
NS b.gtld-servers.net
...
```

---

### Query 2

Ask a `.com` server:

```
What is john.com?
```

The `.com` server replies:

```
NS ns1.cloudflare.com.
NS ns2.cloudflare.com.
```

Again, **no IP address for your website**.

---

### Query 3

Ask `ns1.cloudflare.com`:

```
What is john.com?
```

Now the authoritative server replies:

```
john.com. A     203.0.113.10
john.com. A     203.0.113.11
john.com. AAAA  2001:db8::10
```

These are the records that you created.

---

# Why multiple A and AAAA records?

Suppose your website runs on several servers.

```
Server 1
203.0.113.10

Server 2
203.0.113.11

Server 3
203.0.113.12
```

The DNS zone contains:

```
john.com. A 203.0.113.10
john.com. A 203.0.113.11
john.com. A 203.0.113.12
```

A resolver receives all of them and can choose one (often after the authoritative server has rotated the order), which helps distribute traffic.

Similarly for IPv6:

```
john.com. AAAA 2001:db8::10
john.com. AAAA 2001:db8::11
```

---

# So where are the IP addresses actually stored?

They are stored on the **authoritative name servers** for your domain.

For example:

```
Root Servers
     │
     ▼
.com TLD Servers
     │
     ▼
ns1.cloudflare.com
ns2.cloudflare.com
     │
     ▼
Zone file for john.com

A     203.0.113.10
A     203.0.113.11
AAAA  2001:db8::10
MX    mail.john.com
TXT   ...
```

---

## Summary

* **Root name servers** store only which name servers are responsible for each top-level domain (such as `.com`).
* **TLD name servers** (like the `.com` servers) store the **NS records** that point to the authoritative name servers for `john.com`.
* **Authoritative name servers** for `john.com` store the actual DNS records, including **A**, **AAAA**, **MX**, **TXT**, and others.
* When a DNS lookup returns multiple IPv4 and IPv6 addresses, those multiple **A** and **AAAA** records come from the authoritative name servers for that domain, not from the root or TLD servers.


## How is DNS mapped to your IP

Let's say you choose **Cloudflare DNS** as your authoritative DNS provider, then your DNS records are stored on Cloudflare's globally distributed DNS infrastructure.

For example, if you configure:

```text
john.com.    A      198.51.100.42
www          A      198.51.100.42
```

those records are stored in Cloudflare's DNS database and served by Cloudflare's authoritative name servers (such as `alice.ns.cloudflare.com` and `bob.ns.cloudflare.com`). These servers are replicated across Cloudflare's network so DNS queries are answered quickly from many locations around the world.

## How does this connect to your home server?

Suppose your home Internet connection has a **static public IP**:

```text
Public IP: 203.0.113.50
```

And your web server is running on your PC:

```text
Home PC
192.168.1.20
Port 80 (HTTP)
Port 443 (HTTPS)
```

Your home router is connected to the Internet.

You create an A record in Cloudflare:

```text
john.com.    A    203.0.113.50
```

Now when someone visits `https://john.com`:

### 1. DNS lookup

The browser asks DNS:

```
Where is john.com?
```

Cloudflare replies:

```
203.0.113.50
```

### 2. Browser opens a TCP connection

The browser connects to:

```
203.0.113.50:443
```

### 3. Your router receives the connection

The packet reaches your home router because that is the device with the public IP.

The router might have a port forwarding rule like:

```text
Port 443  → 192.168.1.20:443
```

### 4. Router forwards the traffic

```
Internet
    │
203.0.113.50
    │
Home Router
    │
192.168.1.20
    │
Your web server
```

The web server responds with your website, and the response travels back to the visitor.

---

## Why doesn't DNS know about your computer?

DNS only maps a **name** to an **IP address**.

It doesn't know:

* whether that IP belongs to a home router,
* a cloud VM,
* a Raspberry Pi,
* or a large data center.

Once DNS says "`john.com` → `203.0.113.50`", its job is done. The Internet's routing system takes over to deliver packets to that IP.

---

## What if your ISP changes your IP?

Many home Internet connections use **dynamic IP addresses**. Today your public IP might be:

```text
203.0.113.50
```

Tomorrow it could become:

```text
198.51.100.75
```

In that case, your DNS record would become incorrect unless you update it.

A common solution is **Dynamic DNS (DDNS)**. Software running on your home server or router periodically checks your public IP. If it changes, it uses your DNS provider's API to update the A record automatically.

For example:

```
Old:
john.com → 203.0.113.50

ISP changes IP

DDNS updates Cloudflare

New:
john.com → 198.51.100.75
```

Visitors continue using `john.com`, and DNS points them to your new public IP after the updated record propagates (subject to its DNS cache lifetime).

---

## One more detail: Cloudflare proxy mode

If you're using Cloudflare, there's an optional feature called the **proxy** (shown as the orange cloud in the Cloudflare dashboard).

In that case, DNS no longer returns your home IP directly. Instead:

```
Visitor
    │
Cloudflare Edge Server
    │
(Home Internet)
203.0.113.50
    │
Your Router
    │
Your Web Server
```

The visitor connects to Cloudflare first. Cloudflare then connects to your origin server (your home machine), acting as a reverse proxy. This can provide caching, DDoS protection, TLS termination, and can hide your origin IP from casual observation. If the proxy is disabled (the gray cloud), DNS returns your public IP directly and visitors connect to your router without Cloudflare sitting in the middle.

-------------------------------

# Extra Information  
## What is the Workaround for CGNAT

There are 3 workarounds and all three workarounds solve the same fundamental problem:

> **Your home server cannot receive a new incoming connection because the ISP's CGNAT does not know where to send it.**

The solutions work by creating a path that **you initiate from inside your home network**, because outbound connections are allowed through CGNAT.

---

# 1. Get a public IP from your ISP (the simplest solution)

This is the cleanest approach.

You ask your ISP:

> "Can you provide me with a public IPv4 address?"

The ISP removes you from CGNAT and assigns your router a globally reachable address.

Before:

```text
Internet
    |
203.0.113.50  (ISP CGNAT public IP)
    |
ISP CGNAT
    |
100.64.10.20  (your router)
    |
Your server
```

After:

```text
Internet
    |
203.0.113.75  (your public IP)
    |
Your router
    |
Your server
```

Now you can create a port forwarding rule:

```
Internet TCP port 443
        |
        v
Your router
        |
        v
192.168.1.20:443
```

Someone visiting:

```
https://john.com
```

can reach your server.

### Advantages

✅ Simple
✅ Fast
✅ Works with any protocol (HTTPS, SSH, VPN, game servers)
✅ No extra software needed

### Disadvantages

❌ Often costs extra
❌ Some ISPs don't offer it for home plans
❌ You must secure your own server because it is directly exposed to the Internet

---

# 2. VPN with port forwarding

This is a clever trick.

You rent or use a VPN service that gives you a **public IP address** and allows incoming ports.

Examples of this idea:

```
Your home server
       |
       |  (outgoing VPN connection)
       |
VPN provider server
       |
Public Internet
```

Your home machine creates an outgoing VPN tunnel:

```
Home → VPN server
```

Outbound connections are allowed through CGNAT.

The VPN provider gives you:

```
203.0.113.90
```

Now visitors connect to:

```
https://203.0.113.90
```

The VPN server forwards the traffic through the tunnel:

```
Visitor
   |
203.0.113.90
   |
VPN server
   |
Encrypted tunnel
   |
Your home server
```

---

### Example

Without VPN:

```
Visitor
   |
203.0.113.50
   |
CGNAT
   |
(no route to you)
```

With VPN:

```
Visitor
   |
203.0.113.90
   |
VPN server
   |
existing outgoing tunnel
   |
Your server
```

The important part is that **your server created the tunnel first**.

---

### Advantages

✅ Works behind CGNAT
✅ Your home IP remains hidden
✅ Can expose multiple services
✅ Useful for remote access

### Disadvantages

❌ Requires a VPN provider that supports port forwarding
❌ Extra latency
❌ You depend on the VPN provider

---

# 3. Reverse tunnel (probably the most common modern solution)

A reverse tunnel is similar in concept, but instead of a general VPN, you create a persistent connection to a relay service.

Examples include:

* Cloudflare Tunnel
* SSH reverse tunnels
* Some zero-trust networking products

The idea:

Your home server says:

> "I will create an outgoing connection to the public server. Keep it open."

Example:

```
Home Server
     |
     | outbound connection
     |
Cloud provider
     |
     |
Internet users
```

The provider accepts incoming requests and sends them down the existing tunnel.

---

## Example: Cloudflare Tunnel concept

Normal setup:

```
Visitor
   |
   |
DNS
   |
Your IP
   |
Home Server
```

Problem:

```
Your IP is behind CGNAT
```

With a tunnel:

```
Visitor
    |
    |
Cloudflare Edge
    |
    |
Encrypted tunnel
    |
    |
Home Server
```

Your server runs a small agent:

```
cloudflared
```

It connects outward:

```
Home → Cloudflare
```

Because it is an outbound connection, CGNAT allows it.

When someone visits:

```
https://john.com
```

Cloudflare sends the request through the existing tunnel.

---

### Advantages

✅ Works even behind CGNAT
✅ No port forwarding
✅ Can hide your home IP
✅ Often includes HTTPS automatically
✅ Good security features

### Disadvantages

❌ Depends on a third-party service
❌ Some protocols don't work as easily as HTTP/HTTPS
❌ You need the tunnel software running

---

# Comparing the three

| Method                   | Requires ISP help? | Works behind CGNAT? | Best for                          |
| ------------------------ | ------------------ | ------------------- | --------------------------------- |
| Public IP                | Yes                | N/A                 | Full control, serious hosting     |
| VPN with port forwarding | No                 | Yes                 | Servers, gaming, remote access    |
| Reverse tunnel           | No                 | Yes                 | Websites, APIs, personal projects |

---

# Which would you choose for a home website?

For a small personal website:

```
Domain
   |
Cloudflare DNS
   |
Cloudflare Tunnel
   |
Home Server
```

is usually the easiest.

For a serious production website:

```
Domain
   |
Cloudflare DNS
   |
Cloud VM with public IP
   |
Application server
```

is usually better.

For learning networking and running your own infrastructure:

```
Static public IP
   |
Home router
   |
Your server
```

is the most educational because you see the full Internet path.

The important concept is: **CGNAT blocks unsolicited inbound connections, so every workaround either gives you a real public address or creates an outbound connection that can be used as a path back in.**

------------------
# What is VPN 

A VPN (Virtual Private Network) is basically a way to create a **private, encrypted network connection over the public Internet**.

The easiest way to understand it is to compare a normal Internet connection with a VPN connection.

---

## Normal Internet connection

Without a VPN:

```text
Your laptop
    |
    |  ISP sees:
    |  "User is connecting to google.com"
    |
Home Router
    |
ISP
    |
Internet
    |
Google server
```

Your ISP can see:

* Your public IP address
* Which servers you connect to
* The amount of traffic

(They generally cannot see the contents of HTTPS traffic, because HTTPS is already encrypted.)

---

# With a VPN

A VPN creates an encrypted tunnel between your device and a VPN server.

```text
Your laptop
     |
     |  encrypted tunnel
     |
VPN server
     |
     |
Internet
     |
Website
```

Now websites see the VPN server's IP address, not yours.

Example:

Without VPN:

```text
Website sees:

Source IP:
203.0.113.25   ← your home IP
```

With VPN:

```text
Website sees:

Source IP:
198.51.100.50  ← VPN server IP
```

---

# What actually happens technically?

Suppose you visit:

```text
https://example.com
```

## Step 1: VPN software starts

Your VPN client connects to a VPN server.

For example:

```text
Laptop
192.168.1.10

        |
        |
        v

VPN Server
198.51.100.50
```

They perform authentication and encryption setup.

They agree on encryption keys.

---

## Step 2: A virtual network adapter is created

Your computer creates a virtual interface.

For example:

Before VPN:

```text
Physical adapter:

WiFi
192.168.1.10
```

After VPN:

```text
WiFi
192.168.1.10

VPN adapter
10.8.0.2
```

The VPN adapter behaves like a network card.

---

## Step 3: Your traffic goes into the tunnel

Without VPN:

```text
[Your Laptop] 
      │ 
      ▼
[Your Wi-Fi Router]
      │
      ▼
[ISP's CGNAT] 
      │
      ▼
[Destination Website/Server]
```


With VPN:

```text
[Your Laptop] 
      │ (VPN Client encrypts data)
      ▼
[Your Wi-Fi Router]
      │
      ▼
[ISP's CGNAT] ───► (Sees only an encrypted stream going to a VPN server)
      │
      ▼
[VPN Server] ────► (Decrypts data & assigns its own public IP)
      │
      ▼
[Destination Website/Server]
```


Your ISP sees something like:

```text
Your IP → VPN server IP
```

but the contents are encrypted.

---

# What does the VPN server do?

The VPN server decrypts your packet:

```text
Encrypted packet arrives

VPN server decrypts

Original request:

"Give me example.com"
```

Then it sends the request to the Internet:

```text
VPN server
       |
       |
       v
example.com
```

The website replies:

```text
example.com
       |
       v
VPN server
       |
       v
Encrypted tunnel
       |
       v
Your laptop
```

---

# How does this relate to CGNAT?

This is where VPNs become interesting.

Imagine your home is behind CGNAT:

```text
Your server
     |
Home router
     |
ISP CGNAT
     |
Internet
```

You cannot accept incoming connections.

But you can create an outgoing VPN connection:

```text
Your server
     |
     | outgoing connection
     |
VPN server with public IP
```

Now you have a path:

```text
Internet user
       |
       |
VPN server (public IP)
       |
       |
VPN tunnel
       |
       |
Your home server
```

The VPN server becomes your "front door."

---

# A VPN is not always the same thing

There are two common uses:

## 1. Client VPN (privacy/security)

Example:

```text
Laptop
   |
VPN provider
   |
Internet
```

Used when:

* using public WiFi,
* hiding your home IP,
* accessing another country’s services.

---

## 2. Site-to-site VPN

Used by companies.

Example:

```
Office A
10.1.0.0/16
     |
     |
Encrypted VPN tunnel
     |
     |
Office B
10.2.0.0/16
```

The two networks behave as if they are connected by a private cable.

---

# How is a VPN different from HTTPS?

They solve different problems.

HTTPS:

```text
Browser
   |
encrypted
   |
Website
```

Protects communication **between you and one website**.

VPN:

```text
Your entire device
   |
encrypted tunnel
   |
VPN server
```

Protects your connection from your device up to the VPN server.

You can have both:

```text
Laptop
  |
  | VPN encryption
  |
VPN server
  |
  | HTTPS encryption
  |
Website
```

This is very common.

---

# Common VPN technologies

Some popular VPN protocols:

* WireGuard — modern, lightweight, fast
* OpenVPN — older but widely supported
* IPsec — commonly used in enterprise VPNs

---

A good mental model is:

**A VPN is like drilling a private encrypted pipe through the public Internet.** The Internet still carries the packets, but only the two ends of the pipe can understand what is inside. In the CGNAT case, that pipe is useful because your home machine can create the pipe outward, and then people can use the other end of the pipe to reach you.
