I have this iso image of the linux distribution. And i use 
`sudo dd if="***.iso" of=/dev/sda status=progress bs=4M oflag=sync` 
to write it to a usb drive.

My question is, My computer could be configured for BIOS of UEFI, how does the image work with both?? 

The reason your `dd` command works for both is that modern Linux ISOs are **"Hybrid ISOs."**

The developers of the ISO have carefully laid out the data so that:

- The first 512 bytes contain a valid MBR (for old BIOS systems).
    
- Further into the image, there is a partition table (GPT) that points to a FAT32 image containing the `.efi` files (for UEFI systems).
    
It’s a clever bit of engineering—the same data can be "read" in two different ways depending on how the motherboard decides to look at it.

When you select your USB drive from the boot menu, you are interacting with the **Firmware** (either BIOS or UEFI). Modern computers almost exclusively use **UEFI**, though many have a "Legacy" or "CSM" mode to mimic the old BIOS behavior.

The way UEFI loads that image is fundamentally different—and much more sophisticated—than the old 512-byte MBR trick.

---

### 1. The Death of the "Magic Number" (MBR)

In the old BIOS days, the firmware was "dumb." It just looked at the first sector (Sector 0) of the disk, loaded whatever was there into memory, and hoped it was a bootloader.

**UEFI is "smart."** It doesn't look for code in a specific sector; it looks for **files** in a specific **filesystem**.

### 2. How UEFI Loads the Image

When you plug in your USB and select it in UEFI mode, the firmware follows these steps:

1. **Filesystem Recognition:** UEFI has a built-in driver to read **FAT32** filesystems.
    
2. **The ESP (EFI System Partition):** It searches the USB drive for a partition marked with a specific GUID (the EFI System Partition).
    
3. **The Default Path:** If no specific "boot entry" is defined in the firmware's NVRAM, UEFI looks for a standardized fallback file on the USB:
    
    - `/EFI/BOOT/BOOTX64.EFI` (for 64-bit systems)
        
4. **Execution:** The firmware loads this `.efi` file (which is essentially a special type of executable) into memory and runs it. This file is usually your bootloader (like GRUB, systemd-boot, or the Windows Boot Manager).

The **NVRAM** (Non-Volatile Random Access Memory) is **not** on the USB drive. It is a small physical chip located on your **motherboard**.

Think of NVRAM as the "permanent memory" for your UEFI firmware. Even when the power is cut, it remembers specific settings, like your boot order, date/time, and—most importantly—**Boot Entries**.

---

### How NVRAM and your USB interact

When you use a USB drive, there are two ways the UEFI firmware handles it:

#### 1. The "Removable" (Fallback) Path

This is what happens when you plug in a random Linux ISO.

- The UEFI firmware doesn't know this USB exists until you plug it in.
    
- Because there is no entry for this specific stick in your motherboard's **NVRAM**, the firmware uses its "Automatic Search" logic.
    
- It scans the USB for the standardized path: `/EFI/BOOT/BOOTX64.EFI`.
    
- If it finds that file, it adds a "temporary" entry to your boot menu just for that session.
    

#### 2. The "Registered" Path (Installed OS)

This is what happens **after** you install Linux to your hard drive.

- During the installation of Arch, Void, or Ubuntu, the installer (like `grub-install`) uses a tool called `efibootmgr`.
    
- This tool talks to the motherboard and says: _"Hey, I'm installing 'Void Linux'. Please save this path to your **NVRAM**: `/dev/nvme0n1p1/EFI/void/grubx64.efi`."_
    
- Now, even if you clear your CMOS or change drives, the motherboard "remembers" that "Void Linux" is a boot option because that text string and file path are etched into the **NVRAM chip**.
    

---

### Why does this matter?

**1. The "Missing Boot Entry" Problem:** Sometimes you might reinstall Linux, but your motherboard still shows "Ubuntu" or "Fedora" in the F12 boot menu from three years ago. That’s because those OS installers left "ghost" entries in your **NVRAM**. They aren't on your hard drive; they are stuck on the motherboard chip.

**2. Portability:** If you take your USB drive to a friend's house, it won't be in _their_ NVRAM. That’s why the `/EFI/BOOT/BOOTX64.EFI` file is so important—it’s the "universal" key that allows any UEFI computer to boot the drive without needing a pre-registered NVRAM entry.