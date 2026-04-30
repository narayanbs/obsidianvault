### Use the other one . This one is for more minimal kde with barebones desktop


Installing Arch Linux manually gives you the leanest possible system. By targeting **KDE Plasma 6** specifically with a minimal package selection, you avoid the "bloat" of pre-installed applications like games or office suites.

### 1. Pre-installation

- **Verify Boot Mode:** Ensure you are in UEFI mode (standard for modern PCs).
    
    `ls /sys/firmware/efi/efivars` (If this directory exists, you are in UEFI).
    
- **Connect to Internet:**
    
    - **Ethernet:** Should work automatically.
        
    - **Wi-Fi:** Use `iwctl`.
        
- **Update System Clock:** `timedatectl set-ntp true`
    

---

### 2. Partitioning the Disk

Assuming a single NVMe drive (`/dev/nvme0n1`). Use `cfdisk /dev/nvme0n1` to create:

1. **EFI Partition:** 512MiB (Type: EFI System)
    
2. **Root Partition:** Remaining space (Type: Linux x86-64 root)
    

**Format the partitions:**

Bash

```
mkfs.fat -F 32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

**Mount the partitions:**

Bash

```
mount /dev/nvme0n1p2 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
```

---

### 3. Installation

Install the base system, kernel, and essential firmware.

Bash

```
pacstrap -K /mnt base linux linux-firmware nano
```

---

### 4. Configure the System

Generate fstab and enter the system:

Bash

```
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

**Set Time and Locale:**

Bash

```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
# Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "your-hostname" > /etc/hostname
```

**User and Password:**

Bash

```
passwd  # Set root password
useradd -m -G wheel yourusername
passwd yourusername
# Run 'visudo' and uncomment the line: %wheel ALL=(ALL:ALL) ALL
```

**Bootloader (GRUB):**

Bash

```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

### 5. Install Minimal KDE Plasma 6

This is where you keep it lean. Instead of the `kde-applications` group, we install only the core desktop.

**Drivers and Networking:**

Bash

```
pacman -S networkmanager pipewire pipewire-alsa pipewire-pulse pipewire-jack
systemctl enable NetworkManager
```

**The Plasma 6 Core:**

Bash

```


```

- `plasma-desktop`: The bare minimum desktop environment.
    
- `sddm`: The login manager (Display Manager).
    
- `konsole`: The terminal emulator (essential).
    
- `dolphin`: The file manager.
    

**Enable the Login Manager:**

Bash

```
systemctl enable sddm
```

---

### 6. Finishing Up

Exit the environment and reboot:

Bash

```
exit
umount -R /mnt
reboot
```

### Pro-Tips for "Minimalism"

- **Wayland:** Plasma 6 defaults to Wayland. If you have an NVIDIA GPU, ensure you install `nvidia-utils` and `lib32-nvidia-utils` for the best experience.
    
- **Fonts:** A minimal install might lack good fonts. Run `sudo pacman -S noto-fonts` after logging in to fix any "square boxes" in the UI.
    
- **Browser:** You don't have one yet! Install one with `sudo pacman -S firefox` or `chromium`.
    

Would you like the specific configuration steps for NVIDIA drivers or a swap file to go along with this setup?