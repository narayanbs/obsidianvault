
---

## su and sudo 

At a high level, **`su` and `sudo` both let you run commands as another user (usually root)**, but they do it in very different ways.

---

## 🔑 `su` (switch user)

```bash
su
```

### What it does

-  Switches your session to another user (default: **root** , if none specified or ' - ' is specified)
-  You become that user until you exit
### Authentication
-  Requires the **target user’s password** (usually root’s password)
### Behavior
-  Starts a new shell as that user
-  You stay that user until you type `exit`

### Example 

~~~
su - 
~~~
* prompts for root password
* You become root completely
**Note: it is su dash **

### Key idea

> “I know the root password, so I become root.”

~~~
su alice
~~~
* prompts for alice's password
* You become alice completely

---

## ⚙️ `sudo` (substitute user do)

```bash
sudo command
```

### What it does

- Runs a **single command as another user** (usually root)
-  Can also start a root shell (`sudo -i`)
    
### Authentication

-  Uses **your own password**, not root’s
-  Controlled by `/etc/sudoers`
    
### Behavior

- Only elevates privileges for that command (unless you open a shell)
- Logs what you did (audit trail)
    
### Key idea

> “I’m allowed to act as root for this command.”


### Note
On systems like ubuntu and Gentoo Linux:

- `root` often has a **scrambled/locked password**
- So:
    - `su -` → ❌ fails (no valid root password)
    - `sudo -i` → ✅ works (because of `NOPASSWD`)

---

### sudo -i  gives you a full root shell (login-style)


What’s happening there is less about “logging into” root and more about _temporarily being granted root privileges_ through a controlled mechanism.

On Ubuntu (and most modern Linux systems), the root account **exists**, but it’s typically _locked_—meaning it has no usable password set. So you can’t log in as root directly via something like `su` unless you explicitly set a root password.

Instead, Ubuntu relies on **sudo**.

Here’s how `sudo -i` works under the hood:

**1. You authenticate as yourself, not as root**  
When you run `sudo -i`, the system asks for _your user password_, not root’s. That’s because `sudo` is verifying _your identity_, not root’s.

**2. Permission check via config**  
`sudo` consults the **sudoers configuration file** (usually `/etc/sudoers`). If your user is listed there (often via membership in a group like `sudo`), you’re allowed to execute commands as root.

**3. Privilege escalation**  
Once authenticated and authorized, `sudo` runs the command with root privileges.  
With `-i`, it specifically:


-  Switches to root (`UID 0`)
-  Loads root’s environment (like a full login shell)
-  Drops you into a root shell as if you logged in directly
    

**4. No root password needed**  
Because access is controlled by `sudo`, the root password is unnecessary—and often intentionally disabled for security.

---
### Why this design is used

-  **Accountability**: Actions are tied to your user account (and logged), not a shared root password 
-  **Security**: No need to expose or manage a root password
-  **Granularity**: You can allow specific commands instead of full root access

---

### Quick comparison

- `su -` → requires root password (usually unavailable on Ubuntu)
    
- `sudo command` → runs a single command as root
    
- `sudo -i` → gives you a full root shell (login-style)
---

## sudoers

### When _does_ `/etc/sudoers` get used?

An entry in `/etc/sudoers` matters only when you want to define **who can use `sudo` and how**.

There are two main ways a user gets sudo privileges:

---

### 1. Through group membership (most common on Ubuntu)

Ubuntu already has this line:

```
%sudo ALL=(ALL:ALL) ALL
```

So when you run:

```bash
sudo usermod -aG sudo alice
```

You’re **not editing `/etc/sudoers`**.  
You’re just adding `alice` to the `sudo` group.

Because that group is already defined in `/etc/sudoers`, the user inherits the permissions.

👉 This is the _typical workflow_.

---

### 2. By explicitly editing `/etc/sudoers`

You might manually add a line like:

```
alice ALL=(ALL:ALL) ALL
```

This directly grants `alice` sudo privileges.

This is usually done when:

- You want **fine-grained control**
    
- You don’t want to rely on groups
    
- You’re giving **limited command access**
    

---

### 3. Using `/etc/sudoers.d/` (modular approach)

Instead of editing the main file, you can add a separate file:

```bash
sudo visudo -f /etc/sudoers.d/alice
```

Example content:

```
alice ALL=(ALL) /usr/bin/systemctl
```

This is cleaner and safer for managing multiple users.

---

### So, when is an entry “added”?

An entry in sudo configuration is added only when:

- You manually edit `/etc/sudoers`, or
    
- You create a file in `/etc/sudoers.d/`, or
    
- Some system/package setup script explicitly configures sudo rules
    

**Not** when:

- Creating users
    
- Setting passwords
    
- Logging in
    

---

### Key idea

`sudo` is designed so that:

- **User creation ≠ privilege assignment**
    
- Privileges are **explicitly granted**, not automatic
    

---

The entries in `/etc/sudoers` follow a very specific syntax. They look dense at first, but once you break them into pieces, they’re quite logical.

A typical line you’re referring to is probably something like:

```
%sudo   ALL=(ALL:ALL) ALL
```

Let’s decode that step by step.

---

### 1. `%sudo`

The `%` means this is a **group**, not a user.  
So `%sudo` refers to all users who are members of the `sudo` group.

If it were a single user, it would just be:

```
username ALL=(ALL:ALL) ALL
```

---

### 2. `ALL` (host field)

This specifies **where** the rule applies.

- `ALL` = this rule applies on all hosts
    
- In more complex environments (like shared configs across multiple machines), this could be restricted to specific hostnames
    

---

### 3. `(ALL:ALL)` (run-as specification)

This is one of the most important parts.

Format:

```
(run_as_user:run_as_group)
```

- First `ALL` → you can run commands as **any user**
    
- Second `ALL` → you can run commands as **any group**
    

So this allows:

- `sudo -u root ...`
    
- `sudo -u someuser ...`
    
- `sudo -g somegroup ...`
    

If it were:

```
(ALL)
```

it would mean any user, but default group handling.

---

### 4. `ALL` (command list)

This defines **what commands** are allowed.

- `ALL` = you can run **any command**
    
- You could restrict this, e.g.:
    
    ```
    %sudo ALL=(ALL:ALL) /usr/bin/apt, /usr/bin/systemctl
    ```
    

---

### Putting it all together

```
%sudo ALL=(ALL:ALL) ALL
```

Means:

> Any user in the `sudo` group can run any command, as any user or group, on any host.

---

### Optional tags you might see

Sometimes entries include tags before the command list:

- `NOPASSWD:` → don’t ask for a password
    
    ```
    %sudo ALL=(ALL:ALL) NOPASSWD: ALL
    ```
    
- `PASSWD:` → explicitly require password (default)
    
- `SETENV:` → allow changing environment variables
    

---

### Aliases (to simplify large configs)

The file can define aliases like:

```
User_Alias ADMINS = alice, bob
Cmnd_Alias RESTART = /usr/bin/systemctl restart *
```

Then use them like:

```
ADMINS ALL=(ALL) RESTART
```

---

### Important tip

You should **never edit `/etc/sudoers` directly** with a normal editor.  
Use:

```
sudo visudo
```

It checks syntax before saving, which prevents breaking `sudo` access entirely.

---

## Scrambled Password

**“scrambled password”** means the account’s password has been deliberately made **invalid or unusable**, so no one can log in using a normal password.

On many Linux systems (including Gentoo Linux), user passwords are stored as hashes in files like `/etc/shadow`. If an account has a “scrambled” password, its password field is replaced with something that **can never match any real password** (for example, a random string or special marker like `!` or `*`).

So in your quote:

> “sudo has been configured to run without the need of a password … as both the root and gentoo have a scrambled password”

it implies:

- The `root` and `gentoo` accounts **cannot be logged into using a password**.
    
-  Since there’s no valid password to enter anyway, `sudo` is configured to **not prompt for one**. 
`sudo` is explicitly configured (via `/etc/sudoers`) with something like:
~~~
gentoo ALL=(ALL) NOPASSWD: ALL
~~~
meaning: **run sudo commands without asking for a password**.
    
- This is common in live environments (like a LiveGUI session) where convenience is prioritized over normal authentication.
    

In short: **“scrambled password” = password authentication disabled by making the stored password unusable.**

    
### Creating a scrambled password

“Scrambling” a password just means **locking the account so its stored password hash can never be used for authentication**. You don’t literally scramble it yourself—Linux provides safe commands to do this.

On most Linux distros, here are the standard ways:

---

### 🔒 1. Lock the password (most common)

```bash
sudo passwd -l username
```

- This prepends a `!` to the password hash in `/etc/shadow`.
    
- Result: password login is **disabled** for that user.
    

---

### 🔒 2. Delete the password entirely

```bash
sudo passwd -d username
```

- Removes the password hash.
    
- ⚠️ This can sometimes allow passwordless login depending on config, so it’s less “securely disabled” than locking.
    

---

### 🔒 3. Manually set an invalid hash

```bash
sudo usermod -p '!' username
```

or even:

```bash
sudo usermod -p '*' username
```

- Sets the password field to something that will **never match any real password**.
    

---

### 🔍 What it looks like internally

In `/etc/shadow`, a locked/scrambled account might look like:

```
username:!:...
```

or

```
username:*:...
```

---

### 🧠 Key idea

- A “scrambled password” isn’t a special encryption—it just means:
    
    > **the system is intentionally storing something that can’t be used to authenticate**
    

---



