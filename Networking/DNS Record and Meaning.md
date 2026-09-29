Think of DNS as the Internet’s phonebook: it maps names like `example.com` to information that computers need.

## 1\. A and AAAA Records

#### A record — IPv4 address

 An **A record** maps a domain name to an **IPv4 address**.

 Example:

```
example.com → 93.184.216.34
```

 DNS:

```
example.com.    A    93.184.216.34
```

 When your browser asks:

 > “What IP address belongs to `example.com`?”

 The A record provides the IPv4 address.

 **IPv4:** `93.184.216.34`

---

#### AAAA record — IPv6 address

 An **AAAA record** does the same thing, but for **IPv6**.

 Example:

```
example.com → 2606:2800:220:1:248:1893:25c8:1946
```

 DNS:

```
example.com.    AAAA    2606:2800:220:1:248:1893:25c8:1946
```

 So:

| Record | Purpose     | Address type |
| ------ | ----------- | ------------ |
| A      | Domain → IP | IPv4         |
| AAAA   | Domain → IP | IPv6         |

**Easy way to remember:**\
 `A = IPv4`\
 `AAAA = IPv6`

---

## 2\. PTR Record

 **PTR = Pointer**

 A PTR record is essentially the **reverse of an A/AAAA lookup**.

 Normally:

```
hostname → IP address
```

 A/AAAA:

```
server.example.com → 192.0.2.10
```

 PTR does:

```
192.0.2.10 → server.example.com
```

 This is called **reverse DNS (rDNS)**.

 ### Why is it used?

 One common use is **email servers**.

 Suppose a mail server connects to another mail server from:

```
203.0.113.10
```

 The receiving server can perform a reverse DNS lookup:

```
203.0.113.10 → mail.example.com
```

```
                  A record
mail.example.com ───────────→ 203.0.113.10
       ↑                              │
       │                              │
       └──────── PTR record ──────────┘
```

 This can be one of the checks used when evaluating whether an email server is legitimate.

---

## 3\. CNAME

 **CNAME = Canonical Name**

 A CNAME makes one hostname an **alias of another hostname**.

 Example:

```
www.example.com → example.com
```

 DNS:

```
www.example.com.    CNAME    example.com.
```

 Suppose:

```
example.com → 192.0.2.10
```

 Then:

```
www.example.com
        ↓
   example.com
        ↓
   192.0.2.10
```

 ### Why use CNAME?

 Imagine your application is hosted by a cloud provider:

```
myapp.cloudprovider.com
```

 You want users to access it as:

```
app.example.com
```

 You can create:

```
app.example.com CNAME myapp.cloudprovider.com
```

 This means you don't need to know or maintain the cloud provider's underlying IP address yourself.

 ### Important distinction

 An A record points directly to an IP:

```
app.example.com → 192.0.2.10
```

 A CNAME points to another **hostname**:

```
app.example.com → server.example.net
```

---

## 4\. MX

 **MX = Mail Exchange**

 MX records tell the Internet:

 > **Which mail server should receive email for this domain?**

 For example:

```
example.com.    MX    10    mail.example.com.
```

 Someone sends:

```
alice@example.com
```

 The sending mail server asks DNS:

```
What are the MX records for example.com?
```

 DNS responds:

```
10 mail.example.com
```

 The sending server then looks up:

```
mail.example.com → IP address
```

 and connects to that server to deliver the email.

 ### What does the number mean?

 The number is the **priority**.

 For example:

```
example.com. MX 10 mail1.example.com.
example.com. MX 20 mail2.example.com.
```

 Lower number = higher priority.

 So the sending server tries:

```
mail1 → first
mail2 → if necessary
```

---

## 5\. NS

 **NS = Name Server**

 NS records tell you:

 > **Which DNS servers are authoritative for this domain?**

 For example:

```
example.com.    NS    ns1.example-dns.com.
example.com.    NS    ns2.example-dns.com.
```

 This tells DNS resolvers:

 > “For authoritative DNS information about `example.com`, ask these name servers.”

 ### Why do we need this?

 Suppose you type:

```
www.example.com
```

 A DNS resolver needs to eventually find the authoritative DNS server for `example.com`.

 The NS records help establish:

```
example.com
     ↓
ns1.example-dns.com
ns2.example-dns.com
     ↓
authoritative DNS information
```

 ### A useful distinction

 **NS tells you who is responsible for answering DNS questions for the domain.**

 **A/AAAA tell you the IP address of a hostname.**

---

## 6\. SRV

 **SRV = Service Record**

 SRV records specify **where a particular service is running**. Unlike traditional DNS records (like `A or AAAA`) that only map a domain name to an IP address, an SRV record allows administrators to direct clients to a specific port and hostname where a particular service is running. This is especially useful for protocols that do not run on standard ports or require service discovery.

An SRV record consists of a specific format containing several distinct fields:

- **Service:** The symbolic name of the desired service (e.g., `_sip`, `_ldap`).    
- **Protocol:** The transport protocol used, usually `_tcp` or `_udp`.    
- **Name (Domain):** The domain name for which this record is valid.    
- **Priority:** A number (0–65535) indicating the preference of the target host. **Lower numbers mean higher priority** (clients will try this server first).    
- **Weight:** A relative weight for records with the same priority. Servers with higher weights are chosen more often for load balancing.    
- **Port:** The TCP or UDP port on which the service is running.    
- **Target:** The canonical hostname of the machine providing the service (which then resolves via an `A` or `AAAA` record).

#### General Example

Imagine a company named example.com that uses a Voice over IP (VoIP) service utilizing the Session Initiation Protocol (SIP) over TCP. Instead of forcing users to configure a custom port on their phones, the administrator publishes an SRV record.

The DNS entry looks like this:

    _sip._tcp.example.com. 3600 IN SRV 10 60 5060 sipserver.example.com.

Breakdown of the Example:
*  _ sip.\_tcp: Indicates the SIP service running over TCP.
* 3600: The Time to Live (TTL) in seconds, meaning caching servers can store this record for 1 hour.
* 10 (Priority): High priority. If there were another record with a priority of 20, clients would try this one first.
* 60 (Weight): Used for load balancing if multiple servers share the same priority.
* 5060 (Port): The specific port number where the SIP service is listening (standard SIP port in this case).
* sipserver.example.com. (Target): The actual hostname of the server handling the traffic.

Common Use Cases

* Microsoft Active Directory: Domain controllers rely heavily on SRV records so domain-joined computers can find authentication and directory services.
* XMPP / Jabber: Instant messaging clients use SRV records to find chat servers.
* VoIP / SIP: Finding telephony and signaling servers.
* Email (Autodiscover): Helping mail clients automatically configure IMAP/SMTP settings.

---
## 7\. TXT

 **TXT = Text record**

 TXT records allow arbitrary text to be associated with a domain.

 They are now heavily used for **verification and email security**.

 For example, a domain might have:

```
example.com. TXT "some-verification-value"
```

 A service can ask:

 > “Can you prove that you control `example.com`?”

 You add a specific TXT value to DNS, and the service checks for it.

 ### Common uses

 #### SPF

 SPF information is published using TXT records:

```
example.com. TXT "v=spf1 include:_spf.example.com ~all"
```

 It tells receiving mail servers which systems are authorized to send email for the domain.

 #### DKIM

 DKIM uses DNS TXT records to publish a **public key** that receiving mail servers can use to verify signed email.

 For example:

```
selector1._domainkey.example.com
```

 #### Domain verification

 Services such as cloud platforms may ask you to create:

```
example.com TXT "verification=abc123"
```

 so they can verify domain ownership.

---

 # Putting them all together

 Imagine you own:

```
example.com
```

 Your DNS might look something like:

```
example.com.                  A       203.0.113.10

example.com.                  AAAA    2001:db8::10

www.example.com.              CNAME   example.com.

example.com.                  MX 10   mail.example.com.

mail.example.com.             A       203.0.113.20

example.com.                  NS      ns1.dnsprovider.com.
example.com.                  NS      ns2.dnsprovider.com.

_sip._tcp.example.com.        SRV     10 5 5060 sip.example.com.

example.com.                  TXT     "v=spf1 ..."
```

 Here's what each one means:

| Record | Think of it as | Example |
| --- | --- | --- |
| **A** | Name → IPv4 | `example.com → 203.0.113.10` |
| **AAAA** | Name → IPv6 | `example.com → 2001:db8::10` |
| **PTR** | IP → Name | `203.0.113.10 → example.com` |
| **CNAME** | Alias → Name | `www → example.com` |
| **MX** | Domain → Mail server | `example.com → mail.example.com` |
| **NS** | Domain → DNS servers | `example.com → ns1.dnsprovider.com` |
| **SRV** | Service → Server + Port | `SIP → sip.example.com:5060` |
| **TXT** | Text/metadata | SPF, DKIM, verification, etc. |

### The easiest way to memorize them

```
A      → Where is the server? (IPv4)
AAAA   → Where is the server? (IPv6)
PTR    → Who is this IP?
CNAME  → What other name is this?
MX     → Where should email go?
NS     → Who manages DNS for this domain?
SRV    → Where is this service and port?
TXT    → Extra information/verification
```

 -----------------------------


## Why does the DNS Ptr record point to a different hostname than the forward (A or AAAA) record

When a DNS PTR (Pointer) record for an IP address returns a different hostname than the Forward (A or AAAA) record pointing to that same IP, it usually means the records are out of sync, misconfigured, or serving a specific architectural purpose (like shared hosting).


Yes. The key thing to understand is that **forward DNS and reverse DNS are two separate databases**, even though they involve the same IP address.

 Suppose you have:

```
example.com → 1.2.3.4
```

 That's a **forward lookup**. You're asking:

 > "What IP address does `example.com` point to?"

 Now you ask:

```
1.2.3.4 → ?
```

 That's a **reverse lookup (PTR)**. You're asking:

 > "What hostname does the owner of `1.2.3.4` say this IP belongs to?"

 There is **no requirement that the answers match**.

 ### For example

 Your domain's DNS could contain:

```
A record:

example.com  →  1.2.3.4
```

 But the company that owns `1.2.3.4` might have configured its reverse DNS as:

```
PTR record:

1.2.3.4  →  server123.hostingcompany.com
```

 So you get:

```
Forward:
example.com
    ↓
  1.2.3.4

Reverse:
1.2.3.4
    ↓
server123.hostingcompany.com
```

 This isn't contradictory.

 ### Think of it like a phone book

 Imagine:

 > **John's website** → phone number `555-1234`

 That's like:

 > `example.com` → `1.2.3.4`

 But if you look up the phone number itself, the phone company might have it registered to:

 > **Acme Hosting LLC**

 That's roughly what happens with reverse DNS.

 The important distinction is **who controls each record**:

 | Lookup | Record | Usually controlled by |
| --- | --- | --- |
| `example.com → 1.2.3.4` | A record | Domain owner |
| `1.2.3.4 → hostname` | PTR record | IP address owner/provider |

### Why does this happen so often with servers?

 Imagine you rent a server from a hosting provider.

 They give you:

```
IP: 1.2.3.4
```

 You own/manage:

```
example.com
```

 You configure:

```
example.com → 1.2.3.4
```

 But your hosting provider may initially have:

```
1.2.3.4 → 1-2-3-4.provider.com
```

 So a reverse lookup gives you the provider's hostname rather than `example.com`.

 You can often ask the provider to change the PTR to:

```
1.2.3.4 → mail.example.com
```

 Then you have the desirable arrangement:

```
example.com
     ↓
   1.2.3.4
     ↓
mail.example.com
```

 One subtle point: **the reverse DNS doesn't have to match the exact forward hostname**. `example.com → 1.2.3.4` and `1.2.3.4 → mail.example.com` can both be perfectly valid. What matters is understanding that **A records and PTR records are independently configured**.






