Yes—**for most desktop use on Void Linux, that’s the correct and sufficient core setup for audio using PipeWire**:

```bash
ln -s /etc/sv/pipewire /var/service/
ln -s /etc/sv/wireplumber /var/service/
```

But there are a few important details so you don’t run into “no sound” surprises.

---

# 🎧 What your setup does

- **pipewire** → the audio server (replaces PulseAudio/JACK in most cases)
    
- **wireplumber** → session manager (handles devices, routing, Bluetooth, etc.)
    

Together, they form a complete modern audio stack.

---

# ✅ What you should ALSO ensure

## 1. Install full audio stack

Make sure these are installed:

```bash
xbps-install pipewire wireplumber alsa-pipewire pavucontrol
```

- `alsa-pipewire` → ALSA compatibility layer (very important)
    
- `pavucontrol` → GUI volume/device control
    

---

## 2. Remove conflicts (important)

If PulseAudio is installed, it can interfere:

```bash
xbps-remove pulseaudio
```

(Only do this if it exists on your system.)

---

## 3. Enable ALSA config (recommended)

This ensures apps using ALSA still work:

```bash
echo "pcm.!default pipewire" > ~/.asoundrc
echo "ctl.!default pipewire" >> ~/.asoundrc
```

---

## 4. Reboot or restart services

After enabling:

```bash
reboot
```

or restart manually:

```bash
sv restart pipewire
sv restart wireplumber
```

---

# 🧠 How to check if it works

Run:

```bash
pactl info
```

You should see something like:

- “Server Name: PulseAudio (on PipeWire …)”
    

That means PipeWire is working correctly.

---

# ⚠️ Common missing piece

If you get **no devices or no sound**, it’s usually because:

- `alsa-pipewire` is missing
    
- wireplumber didn’t start
    
- user not in `audio` group
    

Fix user group:

```bash
usermod -aG audio $USER
```

Then log out and back in.

---

# 🎯 Bottom line

✔ Your two commands are the **core requirement**  
✔ But you should also install `alsa-pipewire` and verify no PulseAudio conflict  
✔ Everything else is optional polish

---

If you want, I can also help you set up:

- Bluetooth audio (it needs extra steps on Void)
    
- Low-latency audio for production/music
    
- Or troubleshoot if KDE Wayland doesn’t detect sound devices properly