Good choice—KDE Plasma on Wayland works well on **Void Linux**, but Void won’t set it up for you automatically, so there are a few pieces you need to wire together.

I’ll give you a clean, minimal path that actually works.

---

# 🧩 0. Make sure you’re booted into your installed system

(Not in chroot anymore.)

Update first:

```bash
xbps-install -Su
```

---

# 📦 1. Install KDE Plasma + essentials

Void provides KDE as a meta-package:

```bash
xbps-install kde5 kde5-baseapps
```

For Wayland support:

```bash
xbps-install plasma-wayland-session
```

---

# ⚙️ 2. Install graphics + input essentials

These are critical (people often miss them):

```bash
xbps-install mesa-dri mesa-vulkan-intel mesa-vulkan-radeon mesa-vulkan-intel mesa-vulkan-radeon
```

For Intel/AMD GPUs, Mesa is enough.  
If you use NVIDIA, tell me—steps are different.

Install input + seat management:

```bash
xbps-install elogind dbus
```

Enable services:

```bash
ln -s /etc/sv/dbus /var/service/
ln -s /etc/sv/elogind /var/service/
```

---

# 🔑 3. Install and enable a display manager (SDDM)

KDE uses SDDM:

```bash
xbps-install sddm
```

Enable it:

```bash
ln -s /etc/sv/sddm /var/service/
```

---

# 🌐 4. Network (if not already done)

If you skipped earlier:

```bash
xbps-install NetworkManager
ln -s /etc/sv/NetworkManager /var/service/
```

---

# 🔊 5. Audio (recommended: PipeWire)

```bash
xbps-install pipewire wireplumber
```

Enable:

```bash
ln -s /etc/sv/pipewire /var/service/
ln -s /etc/sv/wireplumber /var/service/
```

---

# 🔤 6. Fonts (otherwise UI looks broken)

```bash
xbps-install dejavu-fonts-ttf
```

---

# 🔄 7. Reboot

```bash
reboot
```

---

# 🖥️ 8. On login screen

- Select **“Plasma (Wayland)”** session in SDDM
    
- Log in
    

---

# ⚠️ Common problems (and fixes)

### ❌ Black screen after login

- Missing `elogind` or `dbus`
    
- GPU driver issue
    

### ❌ Wayland option not showing

- Missing `plasma-wayland-session`
    

### ❌ SDDM not starting

Check:

```bash
sv status sddm
```

---

# 🧠 Notes about KDE on Void

- Void uses **runit**, so services must be manually enabled
    
- Wayland works best on **Intel/AMD GPUs**
    
- NVIDIA works, but requires extra setup (and can be unstable)
    

---

If you want, I can:

- Tailor this for your exact GPU
    
- Help you install a minimal KDE (lighter than full `kde5`)
    
- Or debug if something breaks after reboot
    

Just tell me what hardware you’re on 👍