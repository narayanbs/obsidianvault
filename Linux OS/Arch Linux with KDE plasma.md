Installing Arch Linux with KDE Plasma from a USB drive is very doable, but “complete steps” can’t really be reduced to a simple checklist—you’ll be making choices about partitions, boot mode, and packages along the way. I’ll give you a clear, practical walkthrough that works for most modern systems (UEFI, single disk). If your setup differs, say so and I’ll adjust.

---

# 🧰 1. Prepare the USB (ISO → bootable drive)

1. Download the latest ISO from Arch Linux official site.
    
2. Flash it to a USB (8GB+):
    
    - On Windows: use Rufus
        
    - On Linux:
        
        ```bash
        sudo dd if=archlinux.iso of=/dev/sdX bs=4M status=progress oflag=sync
        ```
        
        _(Replace `/dev/sdX` with your USB device)_
        

---

# 💻 2. Boot into the Arch installer

- Plug in USB → restart → enter BIOS/boot menu
    
- Boot from USB
    
- Choose: **Arch Linux install medium**
    

---

# 🌐 3. Connect to the internet

### Wired:

```bash
ping archlinux.org
```

### Wi-Fi:

```bash
iwctl
device list
station wlan0 connect YOUR_WIFI
exit
```

---

# 🕒 4. Set system clock

```bash
timedatectl set-ntp true
```

---

# 💽 5. Partition the disk

Check disks:

```bash
lsblk
```

Use `cfdisk`:

```bash
cfdisk /dev/sda
```

### Typical UEFI layout:

- 512MB → EFI System Partition (type: EFI)
    
- Rest → Linux filesystem
    

Write changes and quit.

---

# 🧱 6. Format partitions

Example:

```bash
mkfs.fat -F32 /dev/sda1     # EFI
mkfs.ext4 /dev/sda2         # Root
```

---

# 📂 7. Mount partitions

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

---

# 📦 8. Install base system

```bash
pacstrap /mnt base linux linux-firmware nano sudo
```

---

# 🧾 9. Generate fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

# 🔧 10. Chroot into system

```bash
arch-chroot /mnt
```

---

# 🌍 11. Set timezone & locale

```bash
ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
hwclock --systohc
```

Edit locale:

```bash
nano /etc/locale.gen
```

Uncomment:

```
en_US.UTF-8 UTF-8
```

Then:

```bash
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

# 🖥️ 12. Set hostname

```bash
echo "archpc" > /etc/hostname
nano /etc/hosts
```

Add:

```
127.0.0.1 localhost
::1       localhost
127.0.1.1 archpc.localdomain archpc
```

---

# 🔑 13. Set root password

```bash
passwd
```

---

# 👤 14. Create user

```bash
useradd -m -G wheel -s /bin/bash yourname
passwd yourname
EDITOR=nano visudo
```

Uncomment:

```
%wheel ALL=(ALL:ALL) ALL
```

---

# ⚙️ 15. Install bootloader (GRUB)

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# 🌐 16. Install network manager

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

---



### Minimal KDE Plasma (Wayland-first) setup with audio + Bluetooth** 👇

---

# 🧱 1. Base + essentials (inside chroot)

```bash
pacman -S sudo nano networkmanager
systemctl enable NetworkManager
```

---

# 🖥️ 2. Install KDE Plasma (Wayland-ready)

```bash
pacman -S plasma sddm konsole dolphin
```

👉 This installs **KDE Plasma** with Wayland support included by default (Plasma 6+).

Enable login manager:

```bash
systemctl enable sddm
```

---

# 🔊 3. Audio (PipeWire – required)

```bash
pacman -S pipewire pipewire-alsa pipewire-pulse wireplumber
```

This sets up **PipeWire** (Wayland-friendly, modern audio stack).

Optional GUI mixer:

```bash
pacman -S pavucontrol
```

---

# 🔵 4. Bluetooth

```bash
pacman -S bluez bluez-utils bluedevil
systemctl enable bluetooth
```

- Backend: **BlueZ**
    
- KDE GUI: `bluedevil`
    

---

# 🎮 5. GPU drivers (VERY important for Wayland)

### Intel:

```bash
pacman -S mesa vulkan-intel
```

### AMD:

```bash
pacman -S mesa vulkan-radeon xf86-video-amdgpu
```

### NVIDIA (read carefully ⚠️)

```bash
pacman -S nvidia nvidia-utils nvidia-settings
```

👉 Wayland works **only properly on newer NVIDIA (with DRM modeset enabled)**

Add kernel parameter:

```bash
nano /etc/default/grub
```

Find:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

Change to:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet nvidia-drm.modeset=1"
```

Then:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# 🔤 6. Fonts + basic apps

```bash
pacman -S noto-fonts ttf-dejavu firefox ark
```

---

# 🔌 7. Wayland integration fixes (important)

```bash
pacman -S xdg-desktop-portal-kde
```

This ensures:

- file picker works
    
- screen sharing works (Wayland requirement)
    

---

# 🔚 8. Reboot

```bash
exit
umount -R /mnt
reboot
```

---

# 🔐 9. On login screen (important step!)

When SDDM appears:

- Click session selector (bottom corner)
    
- Choose: **“Plasma (Wayland)”**
    

---

# ✅ Verify you're on Wayland

After login:

```bash
echo $XDG_SESSION_TYPE
```

Output should be:

```
wayland
```

---

# ⚠️ Known Wayland caveats

- Screen sharing may need PipeWire + portal (we installed it)
    
- Some older apps still use XWayland (automatic fallback)
    
- NVIDIA can still be slightly less stable vs AMD/Intel
    

---

# 🧠 Minimal package summary

If you want the _cleanest list_, this is basically it:

```bash
pacman -S plasma sddm konsole dolphin \
networkmanager pipewire pipewire-pulse wireplumber \
bluez bluez-utils bluedevil \
mesa vulkan-intel \
xdg-desktop-portal-kde noto-fonts firefox
```

---

# 🚀 Result

You now have:

- ✅ KDE Plasma on Wayland (modern compositor)
    
- ✅ Audio working (PipeWire)
    
- ✅ Bluetooth working
    
- ✅ Minimal, fast system
    

---

If you want, I can help you next with:

- fixing **Wayland stutter/tearing**
    
- setting up **fractional scaling (important on laptops)**
    
- or tuning for **battery life + performance**




### If you want full KDE then 

# 🖼️ 17. Install KDE Plasma

```bash
pacman -S xorg plasma kde-applications sddm
```

Enable display manager:

```bash
systemctl enable sddm
```

_(KDE Plasma is provided by KDE Plasma)_

---

# 🔚 18. Exit and reboot

```bash
exit
umount -R /mnt
reboot
```

Remove USB when prompted.

---

# 🎉 19. First boot

- You’ll see the KDE login screen
    
- Log in with your user account
    
- Done!
    

---

# ⚠️ Common pitfalls

- Wrong disk selected → wipes data
    
- Not using UEFI vs BIOS correctly
    
- Forgetting to enable NetworkManager or SDDM
    
- Missing EFI partition mount
    

---

# 👍 Optional extras (after install)

```bash
pacman -S firefox konsole dolphin git base-devel
```

---

If you want, I can:

- tailor this for **dual boot with Windows**
    
- simplify it using the guided installer (`archinstall`)
    
- or explain _what each step actually does_ so you’re not just copying commands