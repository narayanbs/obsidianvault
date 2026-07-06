
In Linux, a process actually has **multiple user IDs**, each serving a different purpose.

The most important ones are:

* **Real User ID (RUID)**: Who actually started the process.
* **Effective User ID (EUID)**: Whose permissions the kernel uses when checking access to files, devices, etc.

### Real User ID (RUID)

This identifies the user who launched the process.

For example:

```bash
$ id
uid=1000(alice) gid=1000(alice)
```

If Alice runs:

```bash
$ cat file.txt
```

then the process has:

```
RUID = 1000 (alice)
```

---

### Effective User ID (EUID)

Whenever a process tries to do something privileged, like:

* open a file
* bind to a port
* create a directory
* change file ownership

the Linux kernel checks the **effective UID**, **not** the real UID.

Normally:

```
RUID = EUID
```

---

## Example 1: Normal execution

Alice runs:

```bash
$ vim notes.txt
```

```
RUID = alice
EUID = alice
```

Everything is straightforward.

---

## Example 2: Using `sudo`

Suppose Alice runs:

```bash
$ sudo cat /etc/shadow
```

After `sudo` starts the command:

```
RUID = root
EUID = root
```

The new process runs entirely as root (after `sudo` authenticates Alice and creates the new process).

Notice that `sudo` doesn't simply change the EUID of Alice's shell. Instead, it starts a new process with root credentials.

---

## Example 3: Setuid programs (the classic example)

This is where EUID becomes especially important.

Consider:

```bash
$ ls -l /usr/bin/passwd
```

You might see:

```text
-rwsr-xr-x 1 root root ...
```

Notice the **`s`** instead of `x`.

That means the executable has the **setuid** bit set.

Now Alice runs:

```bash
$ passwd
```

The process has:

```
RUID = alice
EUID = root
```

Why?

Because changing a password requires writing to:

```text
/etc/shadow
```

Only root can modify that file.

If `passwd` ran with:

```
EUID = alice
```

it would fail.

Instead:

```
Real UID      = alice
Effective UID = root
```

The kernel allows it to modify `/etc/shadow` because permission checks use the EUID.

---

## Why keep both IDs?

Suppose the `passwd` program wants to know **who is changing their password**.

If it only had:

```
EUID = root
```

it would think root is changing the password.

Instead it can inspect:

```
RUID = alice
```

and know:

> Alice invoked me, but I'm temporarily running with root privileges.

---

## Summary

| Situation                                      | RUID                      | EUID  |
| ---------------------------------------------- | ------------------------- | ----- |
| Alice runs `vim`                               | alice                     | alice |
| Alice runs `sudo vim`                          | root (in the new process) | root  |
| Alice runs a setuid-root program like `passwd` | alice                     | root  |

So your intuition is essentially correct: **the EUID is the user identity whose permissions the process is currently using**. Most of the time it's the same as the real user ID, but mechanisms like **setuid** (and, internally, privilege changes made via system calls such as `seteuid()`) can make them differ. `sudo` is a bit different because it typically launches a **new process** as the target user (often root), rather than just changing the EUID of the existing shell.

### so the kernel will allow alice to change her password, not change someone else's even though the EUID is root


That's the key subtlety: **the kernel does not know it's "Alice changing Alice's password."** It only knows that **the process has EUID = root**, so it allows the operation.

The restriction that Alice can only change **her own** password is enforced by the **`passwd` program**, not by the kernel.

Here's what happens:

1. Alice runs `passwd`.
2. The kernel notices the executable has the **setuid** bit set.
3. The new process starts with:

   ```
   RUID = alice
   EUID = root
   ```
4. When `passwd` needs to update `/etc/shadow`, the kernel checks:

   ```
   EUID == root?
   ```

   Yes → write is allowed.

But before writing, the `passwd` program itself checks who invoked it.

For example, if Alice simply runs:

```bash
passwd
```

`passwd` sees:

```
RUID = alice
```

and updates Alice's password.

If Alice tries:

```bash
passwd bob
```

the program notices:

* RUID = alice
* Target account = bob

Since Alice isn't root, `passwd` rejects the request.

On the other hand, if root runs:

```bash
passwd bob
```

then:

```
RUID = root
EUID = root
```

and the program allows changing Bob's password.

### Why is this secure?

The kernel's job is only to answer questions like:

> "May this process write to `/etc/shadow`?"

It sees:

```
EUID = root
```

and says:

> "Yes."

It does **not** understand higher-level policies like:

* "Users may only change their own password."
* "Only admins may reset someone else's password."

Those policies are implemented by privileged programs like `passwd`, which inspect the real UID (or otherwise determine the caller's identity) before using their elevated privileges.

This separation is a fundamental design principle in Unix:

* **The kernel enforces permissions** (using the EUID and other credentials).
* **Privileged programs enforce application-specific rules** about what actions should be permitted.
