
Installing Void Linux via the manual `chroot` method gives you full control over your system's partitioning and configuration. This guide assumes you have booted the Void Linux live ISO and have an active internet connection.

### Download base iso and create a bootable drive

* Download the glibc live iso image from the website . For ex: `void-live-x86_64-20250202-base.iso`
* Insert the pen drive  and `lsblk -e7`
*  Look for your USB device (e.g.,` /dev/sdb, /dev/sdc`). Do NOT use a partition like `/dev/sdb1` — use the whole device (`/dev/sdb`).
*  Unmount the USB `sudo umount /dev/sdb`
*  Write the ISO to usb
~~~
sudo dd if=path/to/void-live-x86_64-20250202-base.iso of=/dev/sdX bs=4M status=progress oflag=sync
~~~
*  Eject the USB
* I am assuming the computer is connected to the internet using wired ethernet.
* Boot the computer using the USB
*  At the prompt, enter username as `root` and password  `voidlinux`
*  Try pinging `ping voidlinux.org` to ensure internet access is available
###  Installation 
### Part 1: Partitioning and Mounting

First, identify your disk (e.g., `/dev/sda` or `/dev/nvme0n1`) using `lsblk`.

1. **Partition the disk:** Use `cfdisk` to create your partitions. For UEFI systems, you typically need:
   
~~~
   Select the hard disk for partitioning 
(#) lsblk -e7 
(#) cfdisk  /dev/sda
(#) Choose GPT
~~~

**EFI System Partition:** ~1G (Type: `EFI System`)
        
 **_(Optional)_  Swap:  If you need hibernation.

**Root Partition:** The remainder of the disk (Type: `Linux filesystem`)
     
~~~
New: 
	1G
New:
	4G
New:
	Enter twice to select rest of the space 
write: yes
Enter to quit
~~~

        
2. **Format the partitions and mount the filesystem**
~~~
Check the partitions , We have boot partition, a swap partition and a main partition
(#) lsblk 
~~~

Let us create the file system and mount it
~~~
(#) mkfs.ext4 /dev/sda3
(#)mkfs.fat -F 32 /dev/sda1
(#) mkswap /dev/sda2

(#) mount /dev/sda3 /mnt
(#) mount --mkdir /dev/sda1 /mnt/boot/efi
(#) swapon /dev/sda2
~~~
Check again `(#) lsblk 

    Bash
    
    ```
    # Replace sdX with your drive identifier
    mkfs.vfat -F32 /dev/sdX1    # EFI
    mkfs.ext4 /dev/sdX2         # Root
    ```
    
    
---

### Part 2: Installing the Base System

Use the `xbps` package manager to bootstrap the base system. This process downloads the core packages into your target directory.

Bash

```
# Install the base system and essential tools
# Replace x86_64 with your architecture if different (e.g., x86_64-musl)
xbps-install -S -R https://repo-default.voidlinux.org/current -r /mnt base-system linux-firmware nano
```

---

### Part 3: Configuring the System

You must now "enter" your new installation to finalize its settings.

1. **Prepare the Chroot:**
    
    Bash
    
    ```
    (#) xgenfstab  -U  /mnt  >  /mnt/etc/fstab    
    (#) xchroot /mnt /bin/bash
    (#xchroot /mnt ) clear
    ```
    
2. **Basic System Setup:**
    ~~~
	  (#xchroot /mnt ) xbps-install  base-devel  vim  grub  efibootmgr
	  (#xchroot /mnt ) vim /etc/hostname
			zentemple
			:wq
	 (#xchroot /mnt ) vim /etc/default/libc-locales
		uncomment  en_us. UTF-8 . UTF-8
		:wq
	(#xchroot /mnt ) xbps-reconfigure  -f  glibc-locales
	(#xchroot /mnt ) clear
	~~~
    
- **Root password:** `passwd`   let it be `bodhi`
-
- **Create a user:**
```
        useradd -m -G wheel,video,audio,input yourusername
        passwd yourusername
```
        
* ***Install sudo:** `xbps-install -S sudo` and add your user to sudoers (`visudo` and uncomment the `%wheel ALL=(ALL) ALL` line).
~~~
(#xchroot /mnt ) EDITOR=vim visudo
Uncomment the following line 
%wheel  ALL=(ALL:ALL) ALL
	:wq
~~~
        
3. **Bootloader:**
    
    Bash
    
    ```
    xbps-install -S grub-x86_64-efi
    grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=VOID
    xbps-reconfigure -fa
    ```
    

---

### Part 4: KDE Plasma & Wayland Setup (Go to end if you want minimal setup)

Once inside the chroot, you need to install the desktop environment and its dependencies.

1. **Install Essential Packages:**
    
    Bash
    
    ```
    xbps-install -S kde-plasma sddm plasma-wayland-session xorg-server-xwayland NetworkManager
    ```
    
2. **Enable Services:**
    
    Void Linux uses `runit` for service management. You need to link the services to the `/var/service` directory to enable them.


    Bash
    
    ~~~
    # check if elogind is installed 
    xbps-query -Rs elogind
    or
    xbps-query -l | grep elogind
    
    # Enable D-Bus (Required for almost everything in KDE)
	ln -s /etc/sv/dbus /var/service/
	
	# Enable Elogind (CRITICAL - do not skip this)
	ln -s /etc/sv/elogind /var/service/
	
	# Enable Display Manager
	ln -s /etc/sv/sddm /var/service/
	
	# Enable Network Manager
	ln -s /etc/sv/NetworkManager /var/service/
    ~~~
    _Note: `sddm` automatically depends on `elogind`, which is required for power management and session handling._
    
3. **Graphics Drivers:**
    
    Ensure you install the correct drivers for your GPU (e.g., `mesa-dri` for Intel/AMD, or the proprietary driver package for NVIDIA).
    
    Bash
    
    ```
    # Example for Intel/AMD
    xbps-install -S mesa-dri
    ```
    

3. **Networking**

_Note if we install NetworkManager we don't need to do the following config.  See the next section_

Create Config 
~~~
	wpa_passphrase "SSID" "password" > /etc/wpa_supplicant/wpa_supplicant.conf
~~~

 Replace "SSID" with your Wi-Fi network name (exactly as it appears) "Narayan_5G" or "Narayan"
 Replace "password" with your Wi-Fi password
 
You need to enable networking services 
~~~
	ln -s /etc/sv/wpa_supplicant /var/service/
	ln -s /etc/sv/dhcpcd /var/service/	
~~~


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

### Recommendation

- **Use Manual (`wpa_supplicant` + `dhcpcd`)** if you are running a server, a headless system, or a very stripped-down window manager setup where you want to keep the system footprint as small as possible.
    
- **Use NetworkManager** if you are running a laptop or a desktop environment (like GNOME, KDE, or XFCE) and you want your Wi-Fi to "just work" when you switch locations, or if you prefer using `nmcli` and standard desktop networking tools.
    

Once you switch to NetworkManager, you can immediately begin using `nmcli` to manage your connections, exactly as you would on Ubuntu.

---

### Part 5: Finalization

1. **Exit and Reboot:**
    
    Bash
    
    ```
    exit
    umount -R /mnt
    reboot
    ```
    
2. **Start Wayland Session:**
    
    Upon rebooting and logging into SDDM (the graphical login screen):
    
    - Look for the session selector icon (usually at the bottom or corner of the screen).
        
    - Select **"Plasma (Wayland)"**.
        
    - Log in.
        

**Troubleshooting Tips:**

- **Audio:** If you have no sound, install `pipewire` and `wireplumber`, then enable the `pipewire` service.
    
- **Environment Variables:** If some apps (like older Qt apps) don't respect Wayland, you may need to set `QT_QPA_PLATFORM=wayland` in your `/etc/environment` file.
    
- **Service issues:** If SDDM doesn't start, run `sv status sddm` in a terminal to check if it's running or crashing.

## Audio


Because Void Linux follows a minimalist philosophy, it does not assume you want a specific audio server (or any audio server at all). Even though KDE Plasma includes the GUI elements to control volume, those elements cannot "talk" to your hardware without an underlying audio server and session manager running.

To get audio working, you need to install and enable the **PipeWire** stack.

### The Required Audio Stack

You need to install three key packages:

1. **`pipewire`**: The main audio server.
    
2. **`wireplumber`**: The session manager (it handles the actual routing of audio).
    
3. **`pipewire-pulse`**: This is critical. Many applications (and parts of KDE itself) are written to expect a "PulseAudio" server. This package acts as a bridge so those apps can communicate with PipeWire seamlessly.
    

### How to set it up

You can perform these steps while still in your `chroot` environment before you reboot, or after you have finished installing the OS.

#### 1. Install the packages

Bash

```
xbps-install -S pipewire wireplumber pipewire-pulse
```
*Note: pipewire-pulse may be included with pipewire..  so start without pipewire-pulse installation
but include it in configuration*
#### 2. Enable the services

Void Linux uses `runit` for audio services as well. You need to enable them by linking them to `/var/service/`.

Since PipeWire runs as a user-level service in many configurations, the easiest way to ensure it starts correctly for your user is to create the service directory structure. However, in Void, you generally enable the services globally if you want them to start at boot:

Bash

```
# Create the service directory if it doesn't exist (optional, usually exists)
mkdir -p /etc/sv/pipewire
mkdir -p /etc/sv/wireplumber

# If the symlinks aren't automatically created by the package install (they often are, but verify):
ln -s /etc/sv/pipewire /var/service/
ln -s /etc/sv/wireplumber /var/service/
```

_Note: In some newer Void installations, enabling `pipewire` as a root service is handled slightly differently depending on your specific version, but the above is the standard manual way. If you find audio isn't working after a reboot, you can also check `sv status pipewire`._

### A helpful tip for the future

If you ever find that audio is "stuck" or acting strange after a system update, you can usually fix it by restarting the services without a reboot:

Bash

```
sv restart pipewire
sv restart wireplumber
```

By installing these, your KDE volume control applet will suddenly spring to life, and your browser and system sounds will work exactly as you expect!


-----------------------------
## Minimal KDE installation
If you want a **clean KDE Plasma 6 setup on Void Linux**, you can absolutely do it—but there’s one important reality to accept upfront:

👉 A _fully functional_ KDE Plasma session on Void will **effectively require** **elogind**.  
Trying to avoid it leads to missing features (power actions, session tracking, etc.), and Plasma 6 is even more tied to logind behavior than earlier versions.

So the “clean” approach isn’t _no elogind_—it’s **minimal, intentional packages**.

---

# 🔧 Step-by-step: Clean KDE Plasma 6 on Void

## 1. Update base system

```bash
sudo xbps-install -Su
```

---

## 2. Install core KDE Plasma 6 (minimal)

Instead of big meta-packages, install only what you need:

```bash
sudo xbps-install plasma-desktop kde-baseapps kde-cli-tools kde-gtk-config
```

This gives you:

- Plasma shell
    
- Basic KDE apps
    
- Config tools
    

---

## 3. Install essential runtime components

### Display server (Wayland recommended)

```bash
sudo xbps-install kwin wayland
```

### Session + login handling

```bash
sudo xbps-install elogind dbus
```

Enable services:

```bash
sudo ln -s /etc/sv/dbus /var/service
sudo ln -s /etc/sv/elogind /var/service
```

---

## 4. Graphics drivers

Install depending on your hardware:

- Intel:
    
    ```bash
    sudo xbps-install mesa intel-video-accel
    ```
    
- AMD:
    
    ```bash
    sudo xbps-install mesa xf86-video-amdgpu
    ```
    
- NVIDIA:
    
    ```bash
    sudo xbps-install nvidia
    ```
    

---

## 5. Display manager (recommended)

Use **SDDM** (KDE’s default):

```bash
sudo xbps-install sddm
sudo ln -s /etc/sv/sddm /var/service
```

---

## 6. Audio (PipeWire modern setup)

```bash
sudo xbps-install pipewire wireplumber
```

Enable:

```bash
sudo ln -s /etc/sv/pipewire /var/service
sudo ln -s /etc/sv/pipewire-pulse /var/service
```

---

## 7. Fonts + basic usability

```bash
sudo xbps-install noto-fonts-cjk noto-fonts-emoji
```

---

## 8. Reboot and login

After reboot:

- SDDM should start
    
- Choose **Plasma (Wayland)** session
    

---

# 🧼 What makes this “clean”

- No giant `kde5` meta-package
    
- No unnecessary KDE apps (like games, PIM, etc.)
    
- Only core Plasma + essentials
    
- Wayland-first setup
    

---

# ⚠️ If you try to avoid elogind

You _can_ experiment with:

- **seatd**
    
- running Plasma manually via `startplasma-wayland`
    

But expect:

- No GUI shutdown/reboot
    
- Broken power management
    
- Possible session issues
    

In practice, Plasma 6 + Void + no elogind = **not worth the hassle** unless you’re deliberately experimenting.

---

# 👍 Optional minimal extras

Add only what you actually use:

```bash
sudo xbps-install dolphin konsole firefox
```

---

If you want, I can tailor this further (ultra-minimal Wayland-only, no display manager, or tiling setup inside KDE).