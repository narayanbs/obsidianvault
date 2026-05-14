
#### i am curious, when i switch on my computer and the wireless network interface and the router handshake is over, does the kernel fill in the routing table entries each time my system starts

Not exactly “from scratch in the routing table,” but something very close happens each boot.

When your computer starts and the Wi-Fi interface comes up, the kernel itself does **not independently discover the network and populate routes on its own**. Instead, a **userspace network manager + DHCP client** usually drives the process.

Here’s what typically happens:

---

### 1. Interface comes up

When you enable Wi-Fi:

- The kernel loads the driver
    
- The interface (e.g., `wlan0`) is created and marked “up”
    
- At this point, there is usually **no useful IP configuration yet**
    

---

### 2. DHCP handshake (important part)

A userspace service (like NetworkManager, systemd-networkd, or dhclient) sends a **DHCP request** to your router.

The router replies with:

- Your IP address (e.g., `192.168.1.10`)
    
- Subnet mask (e.g., `/24`)
    
- Default gateway (e.g., `192.168.1.1`)
    
- DNS servers
    

---

### 3. Kernel routing table gets updated

Once DHCP provides info, the **userspace DHCP client installs routes into the kernel routing table** via system calls like:

- `netlink` (modern Linux)
    
- older ioctl-based methods
    

So the kernel routing table is **not self-populated**, but **programmatically configured**.

Typical entries added:

- **Connected network route**
    
    ```
    192.168.1.0/24 dev wlan0
    ```
    
- **Default route**
    
    ```
    default via 192.168.1.1 dev wlan0
    ```
    

---

### 4. DNS setup (separate from routing)

DNS entries are usually written to:

- `/etc/resolv.conf` or systemd-resolved
    

Not part of routing table.

---

### 5. Does this happen every time you boot?

Yes—but with nuance:

- The routing table is **recreated each boot**
    
- But it is **not “computed by the kernel”**
    
- It is **reinstalled by networking services after DHCP (or static config)**
    

---

### 6. What the kernel _does_ vs _doesn’t_ do

✔ Kernel:

- Maintains routing table
    
- Forwards packets based on it
    
- Provides APIs to modify it
    

❌ Kernel does NOT:

- Discover gateways
    
- Run DHCP (normally)
    
- Decide default routes automatically
    

---

### One-line summary

At boot, your routing table is **not “filled by the kernel” but configured by userspace networking software after DHCP provides network parameters**.

---
#### so it is dhcpcd which does it

Yes—**if your system uses `dhcpcd`, then it is indeed `dhcpcd` that configures the routing table**, but with an important nuance:

> `dhcpcd` doesn’t “compute routes”; it **receives DHCP info and installs routes into the kernel via netlink/system calls**.

---

### What `dhcpcd` actually does

When it gets a DHCP lease from your router, `dhcpcd` typically:

1. Requests DHCP configuration
    
2. Receives:
    
    - IP address
        
    - Subnet mask
        
    - Default gateway
        
    - DNS servers
        
3. Then it directly configures the system:
    
    - Adds IP address to the interface
        
    - Adds routes to the kernel routing table
        
    - Updates DNS configuration
        

So yes:  
✔ It **writes the default route**  
✔ It **adds the connected subnet route**  
✔ It may also set metrics, MTU, etc.

---

### Example of what it installs

If your router gives:

- IP: `192.168.1.10`
    
- Gateway: `192.168.1.1`
    
- Mask: `/24`
    

`dhcpcd` will typically trigger something equivalent to:

```bash
ip addr add 192.168.1.10/24 dev wlan0
ip route add 192.168.1.0/24 dev wlan0
ip route add default via 192.168.1.1 dev wlan0
```

(Internally done via kernel APIs, not shell commands.)

---

### Important distinction

Even though `dhcpcd` configures routes:

- **Kernel:** stores and uses routing table
    
- **dhcpcd:** _decides what to put there based on DHCP lease_
    

So the flow is:

```
Router (DHCP server)
        ↓
dhcpcd (userspace agent)
        ↓
Kernel routing table (actual data structure used for forwarding)
```

---

### One subtle point

Not all Linux systems use `dhcpcd`:

- NetworkManager may use its own DHCP client
    
- systemd-networkd uses `systemd-networkd-wait-online` + internal DHCP
    
- Some systems use `dhclient`
    

But in all cases, the **pattern is the same**: userspace configures kernel routes.

---

### Bottom line

Yes:  
✔ `dhcpcd` is the component that typically installs routing entries  
❌ The kernel does not do DHCP-based route setup by itself

---
#### but if i have a static ip configured, i need to do it myself?

Yes—**with a static IP setup, you (or your network configuration tool) must define the routing information**, because there’s no DHCP server handing it to you.

But again, you usually don’t manually type kernel commands every time—the system still applies them automatically from config files.

---

## What changes with a static IP?

With DHCP:

- `dhcpcd`/NetworkManager learns:
    
    - IP address
        
    - subnet
        
    - default gateway
        
- then installs routes automatically
    

With static IP:

- there is **no discovery step**
    
- so the system must already know:
    
    - IP address
        
    - subnet mask/prefix
        
    - default gateway (very important)
        

---

## Who actually sets the routes?

Even with static IP, you usually do **not directly edit the kernel routing table**.

Instead, you configure something like:

### 1. systemd-networkd

```ini
[Network]
Address=192.168.1.10/24
Gateway=192.168.1.1
```

### 2. NetworkManager (GUI or CLI)

You set:

- IPv4 address
    
- gateway
    
- DNS
    

### 3. /etc/network/interfaces (older Debian-style)

```ini
iface wlan0 inet static
    address 192.168.1.10
    netmask 255.255.255.0
    gateway 192.168.1.1
```

---

## What happens internally

When the interface comes up:

1. Kernel brings `wlan0` up
    
2. Userspace network service reads your static config
    
3. It then _programs the kernel routing table_
    

Equivalent kernel actions:

```bash
ip addr add 192.168.1.10/24 dev wlan0
ip route add 192.168.1.0/24 dev wlan0
ip route add default via 192.168.1.1
```

So even with static IP:  
✔ routes are still installed automatically  
✔ but from **config**, not DHCP

---

## Important detail people miss

If you set only an IP but forget the gateway:

- You will have only a **local connected route**
    
- You will NOT have internet access
    

Because the kernel has no idea what your:

> “default route to the outside world” is

---

## Key takeaway

- DHCP: routes are learned dynamically (via `dhcpcd`/NetworkManager)
    
- Static IP: routes are predefined in config and applied at boot
    

In both cases:

> The kernel never “figures it out”—it only executes what userspace tells it.

---

If you want, I can show what a _minimal working routing table_ looks like for DHCP vs static and how to break internet connectivity by missing just one route—it’s a great way to make this really intuitive.
