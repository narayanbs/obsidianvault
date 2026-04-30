Here’s a clean step-by-step way to connect to Wi-Fi using **wpa_supplicant + dhcpcd** on a typical Linux system (like Raspberry Pi OS, Arch, minimal Debian setups, etc.).

---

## 1) Find your Wi-Fi interface

Run:

```bash
ip link
```

Look for something like:

- `wlan0` (most common)
    
- or `wlp2s0`
    

I’ll assume `wlan0` below.

---

## 2) Create Wi-Fi config file

Generate it using:

```bash
wpa_passphrase "YourSSID" "YourPassword" > /etc/wpa_supplicant/wpa_supplicant.conf
```

Then (important):

```bash
chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf
```

---

## 3) Start wpa_supplicant

Run it in the background:

```bash
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

Meaning:

- `-B` → run in background
    
- `-i wlan0` → interface
    
- `-c` → config file
    

---

## 4) Get an IP address (DHCP)

Now request an IP using dhcpcd:

```bash
dhcpcd wlan0
```

If successful, your system gets an IP from the router.

---

## 5) Verify connection

Check IP:

```bash
ip a show wlan0
```

Test internet:

```bash
ping -c 3 google.com
```

---

## 6) (Optional but useful) Make it automatic on boot

Enable services:

```bash
systemctl enable wpa_supplicant@wlan0
systemctl enable dhcpcd
```

Then start them:

```bash
systemctl start wpa_supplicant@wlan0
systemctl start dhcpcd
```

---

## Common issues (quick fixes)

### ❌ No IP assigned

Try restarting dhcpcd:

```bash
dhcpcd -n wlan0
```

### ❌ Authentication failed

Check:

- SSID/password correctness
    
- hidden spaces in SSID
    
- router security mode (WPA2/WPA3 issues)
    

### ❌ Interface blocked

Run:

```bash
rfkill unblock wifi
```

---
### Ubuntu specific 

On **Ubuntu**, you usually don’t need to manually manage `wpa_supplicant` unless you’re on a minimal/server install. Ubuntu typically uses **NetworkManager**, which is much simpler.

But since you’re asking for the manual `wpa_supplicant + dhcpcd` method, here’s the **Ubuntu-appropriate way** (and also the better NetworkManager option).

---

# ✅ Recommended (Ubuntu Desktop / Server with NetworkManager)

## 1) Connect via nmcli (easiest)

List Wi-Fi networks:

```bash
nmcli dev wifi list
```

Connect:

```bash
nmcli dev wifi connect "YourSSID" password "YourPassword"
```

Check connection:

```bash
nmcli dev status
```

That’s it—no config files needed.

---

# 🛠️ Manual method (wpa_supplicant + dhcpcd on Ubuntu)

Use this only if:

- You’re on minimal Ubuntu server
    
- NetworkManager is disabled
    
- You want low-level control
    

---

## 1) Install required packages

```bash
sudo apt update
sudo apt install wpasupplicant dhcpcd5
```

---

## 2) Find your Wi-Fi interface

```bash
ip link
```

Look for `wlan0` or `wlp2s0`.

---

## 3) Create Wi-Fi config

```bash
sudo wpa_passphrase "YourSSID" "YourPassword" | sudo tee /etc/wpa_supplicant.conf > /dev/null
```

Secure it:

```bash
sudo chmod 600 /etc/wpa_supplicant.conf
```

---

## 4) Connect to Wi-Fi

```bash
sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant.conf
```

---

## 5) Get IP address

```bash
sudo dhcpcd wlan0
```

---

## 6) Verify

```bash
ip a show wlan0
ping -c 3 google.com
```

---

# ⚠️ Important Ubuntu note

On Ubuntu, NetworkManager may interfere with manual setup.

If you get conflicts, temporarily stop it:

```bash
sudo systemctl stop NetworkManager
```

To re-enable later:

```bash
sudo systemctl start NetworkManager
```

---

# 👍 What you should use

- **Ubuntu Desktop → use `nmcli` (best)**
    
- **Ubuntu Server → usually still NetworkManager or netplan**
    
- **Only use wpa_supplicant manually if troubleshooting or minimal system**
    

---
### Ubuntu Desktop 

On **Ubuntu Desktop**, you should almost never use `wpa_supplicant` or `dhcpcd` manually. The system is designed to handle Wi-Fi through **NetworkManager**, and that’s the cleanest and most reliable way.

Here’s how you should do it:

---

# ✅ Easiest way (Ubuntu Desktop)

## Option 1: GUI (normal method)

1. Click the **Wi-Fi icon** (top-right corner)
    
2. Select your network (SSID)
    
3. Enter password
    
4. Done
    

---

## Option 2: Terminal (recommended if you prefer CLI)

### 1. List available networks

```bash
nmcli dev wifi list
```

### 2. Connect to Wi-Fi

```bash
nmcli dev wifi connect "YourSSID" password "YourPassword"
```

That’s it. Ubuntu will:

- configure `wpa_supplicant` internally
    
- get IP automatically via DHCP
    
- reconnect on reboot
    

---

# 🔍 Check connection status

```bash
nmcli dev status
```

or

```bash
ip a
```

---

# 🔧 Useful troubleshooting commands

### Restart networking

```bash
sudo systemctl restart NetworkManager
```

### Turn Wi-Fi on/off

```bash
nmcli radio wifi on
nmcli radio wifi off
```

### Show saved connections

```bash
nmcli connection show
```

---

# ⚠️ Important advice

Since you're on Ubuntu Desktop:

- ❌ Avoid manually editing `/etc/wpa_supplicant.conf`
    
- ❌ Avoid `dhcpcd`
    
- ❌ Avoid manually starting `wpa_supplicant`
    

Those are mainly for minimal/server setups and can conflict with NetworkManager.

---

If you want, I can also show you how to:

- auto-connect to hidden networks
    
- connect without typing password each time (securely saved)
    
- fix “Wi-Fi connected but no internet” issues on Ubuntu
    

Just tell me 👍

### Systemd trivia

~~~
systemctl status wpa_supplicant@wlan0
systemctl status dhcpcd
~~~

 Running those commands will ensure that `wpa_supplicant` and `dhcpcd` start automatically whenever your system reboots.

### What these commands do
When you run `systemctl enable <service>`, you are creating a symbolic link in the system's configuration directory. This tells the system manager (`systemd`) to automatically load these services during the boot sequence.



### A Pro-Tip: Using `--now`
While `enable` sets the services to start on the *next* boot, it does not actually start them right now. To enable them for future reboots **and** start them immediately without needing to manually run `systemctl start`, you can use the `--now` flag:

```bash
systemctl enable --now wpa_supplicant@wlan0
systemctl enable --now dhcpcd
```

### Checklist for Success
For these services to actually connect your network, ensure the following are in place:

* **Configuration Files:** Ensure you have a valid configuration file for `wpa_supplicant` (typically located at `/etc/wpa_supplicant/wpa_supplicant.conf` or similar, depending on your distribution). Without the correct SSID and password configured there, the service will start but fail to connect.
* **Dependency Check:** `dhcpcd` usually requires an interface to be "up" before it can request an IP address. `wpa_supplicant` handles the authentication for the WiFi, so it acts as a prerequisite for `dhcpcd` in most wireless setups.
* **Verification:** After running your commands, you can check their current status with:
    ```bash
    systemctl status wpa_supplicant@wlan0
    systemctl status dhcpcd
    ```
    If you see **"active (running)"** in green text, your configuration is working correctly.

### Wpa_supplicant and iwd

So you are saying the NetworkManager internally uses iwd in latest versions of Ubuntu rather than wpa_supplicant and the configuration files are internal to NetworkManager

That is a very common point of confusion, but to clarify: **Ubuntu does not use `iwd` by default.** It still uses `wpa_supplicant` as the default backend for `NetworkManager`.

While `iwd` is a newer, more modern technology, `wpa_supplicant` remains the standard on most major Linux distributions, including Ubuntu.

Here is the reality of how these pieces fit together:

### 1. NetworkManager is the "Boss"

You are absolutely correct that the configuration files are internal to `NetworkManager`. Think of `NetworkManager` as the manager and `wpa_supplicant` (or `iwd`) as the worker.

- **You** (or the GUI) talk to `NetworkManager`.
    
- **NetworkManager** stores the connection settings (SSID, password, security type) in `/etc/NetworkManager/system-connections/`.
    
- **NetworkManager** then takes that information and "feeds" it to the worker (`wpa_supplicant`).
    

Because `NetworkManager` is the one feeding the configuration to the worker, you should **never** try to edit `wpa_supplicant.conf` or `iwd` settings directly. `NetworkManager` will simply ignore your changes, or worse, overwrite them the next time it decides to "refresh" the connection.

### 2. Why you couldn't find the `wpa_supplicant` config

You likely couldn't find a `wpa_supplicant.conf` file because:

- **Dynamic Generation:** When `NetworkManager` is running, it often manages the supplicant dynamically. It creates and maintains its own temporary configuration or communicates with the supplicant via a D-Bus API, rather than relying on a static, user-editable text file like you would in a "manual" setup.
    
- **Abstraction:** The goal of `NetworkManager` is to hide the complexity of the backend from you. It doesn't want you to know (or care) if it's using `wpa_supplicant` or `iwd`—it just wants to make sure your Wi-Fi works.
    

### 3. Can you switch to `iwd`?

Yes, you _can_ switch Ubuntu to use `iwd` as the backend instead of `wpa_supplicant`, but it is an "advanced" move that requires manual configuration of `NetworkManager`.

If you did that, you would create a file like `/etc/NetworkManager/conf.d/iwd.conf` and tell `NetworkManager` to use `wifi.backend=iwd`. **But for standard Ubuntu usage, you don't need to do this.** The performance difference on a desktop is usually negligible, and staying with the default (`wpa_supplicant`) ensures your system remains in the state that Ubuntu updates and support channels expect.

**Summary for your records:**

- **`NetworkManager` (Ubuntu default):** Controls everything. Configuration is in `/etc/NetworkManager/system-connections/`.
    
- **`wpa_supplicant` (Ubuntu default backend):** The silent worker. Do not touch its config.
    
- **`iwd` (Alternative backend):** An optional, modern alternative. Not the default on Ubuntu.
    

If you are ever troubleshooting, stick to `nmcli` or the GUI. If those tools fail, the issue is almost always a driver issue, a blocked radio switch, or a configuration error inside `NetworkManager`'s own profiles—not a problem with how `wpa_supplicant` is configured.

#### Is networkmanager available in void linux or do we explicitly configure wpa_supplicant and dhcpd

Yes, **NetworkManager is absolutely available in Void Linux.** You are not forced to manually configure `wpa_supplicant` and `dhcpcd`, although that is the "classic" or "minimalist" way Void is often set up.

If you prefer the convenience of `nmcli` or GUI tools (like `nm-applet`) and want to stop manually editing config files, you can easily switch to NetworkManager.

### How to set up NetworkManager in Void Linux

Since Void uses `runit` for service management, you can replace your manual network configuration with NetworkManager by following these steps:

#### 1. Install the packages

First, ensure you have NetworkManager installed:

Bash

```
sudo xbps-install -S NetworkManager
```

#### 2. Enable D-Bus

NetworkManager relies on the D-Bus system bus to communicate. You must ensure `dbus` is enabled and running:

Bash

```
sudo ln -s /etc/sv/dbus /var/service/
```

#### 3. Start NetworkManager

Now, symlink the NetworkManager service to your `/var/service/` directory (the standard Void way to start services):

Bash

```
sudo ln -s /etc/sv/NetworkManager /var/service/
```

#### 4. Disable your old services (CRITICAL)

If you previously had `wpa_supplicant` or `dhcpcd` running via `runit`, **you must disable them.** If you leave them running, they will conflict with NetworkManager, leading to a broken network connection.

Bash

```
# Stop and remove the symlinks for your old network services
sudo rm /var/service/wpa_supplicant
sudo rm /var/service/dhcpcd
```

### Comparison: Manual vs. NetworkManager in Void

|**Feature**|**Manual (wpa_supplicant + dhcpcd)**|**NetworkManager**|
|---|---|---|
|**Complexity**|Higher (manual config files)|Lower (automated)|
|**Control**|Absolute (you define the behavior)|High (handled by daemon)|
|**GUI Support**|Very limited|Excellent|
|**Setup Speed**|Fast for single networks|Fast for roaming/multiple networks|
|**Dependency**|Minimal (lightweight)|Requires D-Bus and more resources|

### Recommendation

- **Use Manual (`wpa_supplicant` + `dhcpcd`)** if you are running a server, a headless system, or a very stripped-down window manager setup where you want to keep the system footprint as small as possible.
    
- **Use NetworkManager** if you are running a laptop or a desktop environment (like GNOME, KDE, or XFCE) and you want your Wi-Fi to "just work" when you switch locations, or if you prefer using `nmcli` and standard desktop networking tools.
    

Once you switch to NetworkManager, you can immediately begin using `nmcli` to manage your connections, exactly as you would on Ubuntu.

### Why should i enable dbus?? 

Enabling **D-Bus** (Desktop Bus) is crucial because it acts as the **"central nervous system"** for modern Linux desktop communications. Without it, NetworkManager cannot receive commands from your user-level tools or notify your desktop environment of network changes.

Here is why it is technically necessary for NetworkManager:

### 1. Inter-Process Communication (IPC)

NetworkManager runs as a system-level daemon (usually as `root`) to manage hardware devices and IP configurations. However, tools like `nmcli`, `nmtui`, or your GUI system tray icon (like GNOME or KDE network applets) run as your normal user.

Because these processes live in different "memory silos" for security and stability, they cannot talk to each other directly. D-Bus provides a standardized "message bus" where these processes can exchange information securely.

### 2. The "Bridge" Between User and Root

When you run `nmcli connection up`, the following chain of events happens via D-Bus:

1. **The Request:** `nmcli` sends a message onto the D-Bus: _"Hey, NetworkManager, please activate this connection."_
    
2. **The Authorization:** D-Bus checks security policies (PolicyKit) to ensure your user has permission to do this.
    
3. **The Execution:** NetworkManager (listening on D-Bus) receives the message, parses it, and performs the necessary hardware-level actions (talking to the kernel, configuring the interface).
    
4. **The Response:** NetworkManager sends a message back through D-Bus: _"I have successfully connected to the network."_
    

Without D-Bus, `nmcli` would have no way to reach the NetworkManager daemon to tell it what to do.

### 3. Asynchronous Events

NetworkManager needs to tell your desktop environment about things happening in the background, such as:

- "I just detected a new Wi-Fi network."
    
- "I lost the internet connection."
    
- "I am currently obtaining an IP address."
    

NetworkManager emits these as "signals" on the D-Bus. Your desktop environment (like GNOME or XFCE) is constantly "listening" to these signals so that it can update the network icon in your taskbar in real-time. If D-Bus isn't running, your desktop tray will likely show a "Network Disconnected" icon forever, even if you are connected, because the desktop applet never received the "Connected" signal.

### Why is this specific to Void Linux?

In distributions that use `systemd` (like Ubuntu or Fedora), D-Bus is often started automatically as a dependency of the system.

In **Void Linux**, the system is minimalist by design. `runit` only starts exactly what you tell it to start. Because D-Bus is an independent service, you must explicitly tell `runit` to start the `dbus` service (`ln -s /etc/sv/dbus /var/service/`) before NetworkManager can function. If you don't, NetworkManager will start, but it will essentially be "deaf and mute"—unable to talk to any of your tools.

**In summary:** Think of NetworkManager as a skilled engineer, and D-Bus as the radio they use to communicate with the rest of the building. Without the radio, the engineer is working, but they cannot receive instructions or report on their progress.