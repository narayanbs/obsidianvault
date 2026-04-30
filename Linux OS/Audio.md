### PipeWire and ALSA

PipeWire and ALSA are both parts of the Linux audio stack, but they operate at different layers and solve different problems.

ALSA (Advanced Linux Sound Architecture) is the low-level audio system in the Linux kernel. It provides the actual drivers and interfaces that let the operating system talk to your sound hardware (sound cards, onboard audio, USB audio devices). In simple terms, ALSA is what directly controls the audio device.

PipeWire, on the other hand, is a higher-level userspace media server. It sits above ALSA and manages audio (and also video streams) for applications. Instead of applications talking directly to ALSA, they usually talk to PipeWire, which then routes audio through ALSA to the hardware.

### How they relate in practice

- **ALSA = hardware layer**
    
    - Talks to sound devices directly (kernel space)
        
    - Provides device drivers and basic audio I/O
        
    - No advanced routing, mixing, or per-app control by itself
        
- **PipeWire = audio manager / router**
    
    - Runs in user space
        
    - Handles mixing, routing, permissions, low-latency processing
        
    - Presents virtual audio devices to applications
        
    - Uses ALSA underneath to actually send/receive sound from hardware
        

### The pipeline looks like this:

Applications → PipeWire → ALSA → Sound hardware

### Why this design exists

Historically, Linux had multiple competing audio systems:

- ALSA (still the foundation)
    
- PulseAudio (for desktop audio mixing)
    
- JACK (for professional low-latency audio)
    

PipeWire was designed to unify these roles. It can:

- Replace PulseAudio for desktop audio
    
- Replace JACK for pro audio workflows
    
- Still rely on ALSA for hardware access


### How they interact: The "ALSA Plugin"

This is the most important part of the relationship. To make sure old applications still work, PipeWire provides an **ALSA Compatibility Plugin** (part of `pipewire-alsa`).

1. **Modern Apps:** Know how to talk to PipeWire directly.
    
2. **Legacy Apps:** Are "hardcoded" to only talk to ALSA. They don't know PipeWire exists.
    
3. **The Magic:** The ALSA compatibility plugin intercepts the calls from those legacy apps. It acts as a "translator" that takes the data the app _thought_ was going to the hardware and redirects it straight into PipeWire’s routing system.


###  Old Audio Stack
~~~
Apps (desktop) → PulseAudio → ALSA → hardware

Apps (pro audio) → JACK → ALSA → hardware
~~~
###  Modern stack with pipewire
~~~
Apps (all types) → PipeWire → ALSA → hardware
~~~

### Key takeaway

ALSA is the **low-level driver layer**, while PipeWire is the **modern audio server that manages everything above ALSA** and uses it to interact with real sound hardware.

### Checking on Ubuntu

Alright, let’s check what your system is actually using 👍

Open a terminal and run:

```
pactl info | grep "Server Name"
```

### What to look for:

* If you see **`PulseAudio (on PipeWire ...)`** → you’re using **PipeWire** (with compatibility layer)
* If you see just **`PulseAudio`** → you’re still on **PulseAudio**

---

### Alternative check (more direct)

Run:

```
systemctl --user status pipewire
```

* If it says **active (running)** → PipeWire is in use
* If not → probably still PulseAudio

---

### Bonus: check both at once

```
ps aux | grep -E "pipewire|pulseaudio"
```

---


