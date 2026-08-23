`ss -lntup` shows **network sockets that are currently listening**, along with the protocols and processes using them.

The options mean:

* **`-l`** → **listening** sockets only
* **`-n`** → show **numeric** addresses and port numbers; don't resolve names
* **`-t`** → show **TCP** sockets
* **`-u`** → show **UDP** sockets
* **`-p`** → show the **process/program** using the socket

So:

```bash
ss -lntup
```

roughly means:

> **"Show me all listening TCP and UDP ports, using numeric addresses/ports, and tell me which processes own them."**

You might get something like:

```text
Netid State   Local Address:Port  Peer Address:Port  Process
tcp   LISTEN  0.0.0.0:22         0.0.0.0:*          users:(("sshd",pid=1234))
tcp   LISTEN  127.0.0.1:631      0.0.0.0:*          users:(("cupsd",pid=5678))
udp   UNCONN  0.0.0.0:68         0.0.0.0:*          users:(("dhclient",pid=901))
```

For your previous `/etc/services` question, you could use:

```bash
ss -lntup | grep ':7 '
```

to see whether something is actually listening on **port 7**.

One important distinction: **`/etc/services` tells you what a port is conventionally assigned to; `ss` tells you what is actually listening on your machine right now.**
