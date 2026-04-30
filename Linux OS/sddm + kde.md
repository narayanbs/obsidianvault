

Here’s how to get **SDDM + KDE (Plasma Wayland)** working on Void Linux.

---

# 1. Make sure required packages are installed

You likely already have KDE, but double-check:

```sh
sudo xbps-install -S kde5 sddm
```

For Wayland support (important):

```sh
sudo xbps-install -S plasma-wayland-session
```

---

# 2. Enable SDDM (this is the key step)

Void uses **runit**, not systemd. So you enable services like this:

```sh
sudo ln -s /etc/sv/sddm /var/service/
```

That’s what makes SDDM start at boot.

---

# 3. (Optional but recommended) Enable dbus

KDE needs D-Bus to function properly:

```sh
sudo ln -s /etc/sv/dbus /var/service/
```

---

# 4. Reboot

```sh
sudo reboot
```

---

# What should happen

After reboot:

- SDDM login screen appears
    
- You can select session:
    
    - **Plasma (Wayland)** (preferred)
        
    - Plasma (X11) (fallback)
        
- Log in → KDE starts via KWin
    

---

# If SDDM doesn’t show up

Check service status:

```sh
sv status sddm
```

If it’s not running, check logs:

```sh
sudo cat /var/log/sddm.log
```

---

# Common pitfalls (Void-specific)

- Missing GPU drivers (especially for Wayland)
    
- Not enabling `dbus`
    
- Installing KDE without the Wayland session package
    
- Running inside a VM without proper graphics acceleration
    

---

# Minimal fallback (if you get stuck)

You can still launch KDE manually from TTY:

```sh
startplasma-wayland
```

---


Good—Intel GPU is the _least painful_ case for KDE on Wayland. You’re in the “it should just work” category, assuming drivers and services are set correctly.

Here’s what you need on **Void Linux + Intel + KDE + SDDM + Wayland**.

---

# 1. Intel graphics drivers (important but usually already present)

Make sure you have Mesa installed:

```sh
sudo xbps-install -S mesa-dri vulkan-loader intel-video-accel
```

Optional but useful:

```sh
sudo xbps-install -S mesa-vulkan-intel
```

This enables:

- OpenGL acceleration
    
- Vulkan (for newer KDE effects and apps)
    
- Proper Wayland rendering
    

---

# 2. Enable required services (Void runit)

These are mandatory for KDE login via SDDM:

### SDDM (login screen)

```sh
sudo ln -s /etc/sv/sddm /var/service/
```

### D-Bus (required by KDE)

```sh
sudo ln -s /etc/sv/dbus /var/service/
```

---

# 3. Make sure KDE Wayland session is installed

```sh
sudo xbps-install -S plasma-desktop plasma-wayland-session
```

This gives you the Wayland session option in SDDM.

---

# 4. Enable elogind (important on Void)

KDE expects session tracking:

```sh
sudo ln -s /etc/sv/elogind /var/service/
```

Without this, you may get login loops or broken permissions.

---

# 5. Reboot

```sh
sudo reboot
```

---

# 6. What you should see

After reboot:

- SDDM login screen appears
    
- You choose session:
    
    - **Plasma (Wayland)**
        
- KDE starts using KWin
    
- Intel GPU handles rendering via Mesa
    

---

# If something goes wrong

### Black screen after login

Check logs:

```sh
journalctl -b
```

or SDDM logs:

```sh
cat /var/log/sddm.log
```

### Wayland session missing

Install:

```sh
sudo xbps-install -S plasma-wayland-session
```

### Fallback option (always works)

```sh
startplasma-x11
```

---

# Intel + KDE Wayland reality check

With Intel GPUs:

- Wayland support is generally stable
    
- No proprietary drivers needed
    
- Occasional issues only on very old Intel chips (pre-HD 4000)
    

---

If you want, I can also help you trim KDE down for a lighter Void setup (SDDM + Plasma without extra KDE apps), or help you switch between Wayland and X11 cleanly.