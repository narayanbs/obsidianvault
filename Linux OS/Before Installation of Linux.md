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

When you turn on your computer, the UEFI firmware checks its NVRAM entries first. This is why your internal hard drive boots directly into your existing OS by default.

When you force the computer to look at the USB drive, the firmware checks the USB's NVRAM entries (which usually don't exist for a portable drive) and  follows these steps:

1. **Filesystem Recognition:** UEFI has a built-in driver to read **FAT32** filesystems.
    
2. **The ESP (EFI System Partition):** It searches the USB drive for a partition marked with a specific GUID `C12A7328-F81F-11D2-BA4B-00A0C93EC93B` (the EFI System Partition).
    
3. **The Default Path:**  The firmware looks inside the root directory of that partition for a folder named `EFI` containing a folder named `BOOT`, containing the file `BOOTX64.EFI`.
    
    - `/EFI/BOOT/BOOTX64.EFI` (for 64-bit systems)
        
4. **Execution:** The firmware loads this `.efi` file (which is essentially a special type of executable) into memory and runs it. This file is usually your bootloader (like GRUB, systemd-boot, or the Windows Boot Manager).


The **NVRAM** (Non-Volatile Random Access Memory) is **not** on the USB drive. It is a small physical chip located on your **motherboard**.

Think of NVRAM as the "permanent memory" for your UEFI firmware. Even when the power is cut, it remembers specific settings, like your boot order, date/time, and—most importantly—**Boot Entries**.

---

This is what happens **after** you install Linux to your hard drive.

- During the installation of Arch, Void, or Ubuntu, the installer (like `grub-install`) uses a tool called `efibootmgr`.
    
- This tool talks to the motherboard and says: _"Hey, I'm installing 'Void Linux'. Please save this path to your **NVRAM**: `/dev/nvme0n1p1/EFI/void/grubx64.efi`."_
    
- Now, even if you clear your CMOS or change drives, the motherboard "remembers" that "Void Linux" is a boot option because that text string and file path are etched into the **NVRAM chip**.
    

---

# How does the image manage hardware on the new system?

When you boot into a Linux live USB on a brand-new computer, the firmware (UEFI or BIOS) hands over control to the Linux kernel. From that point on, **the Linux kernel takes over hardware management entirely**, bypassing the firmware for most runtime operations.

Here is how tools like  `fdisk` ultimately gets the list of block devices:

### 1. The Kernel Probes the Hardware

As the Linux kernel boots up, it scans the computer's buses (PCI, PCIe, USB, SATA, NVMe) looking for attached storage controllers and drives. It loads the appropriate device drivers (kernel modules) for whatever hardware it finds—whether it's an NVMe SSD, a SATA hard drive, or the USB stick you booted from.

### 2. The Kernel Creates Device Nodes

Once the kernel detects a storage drive, it registers it as a block device and creates corresponding entries in virtual filesystems managed by the kernel:

* **`/dev/`**: The kernel populates device nodes (like `/dev/sda`, `/dev/nvme0n1`, etc.) which act as file-like interfaces to interact with the raw hardware.
* **`/sys/block/`**: The kernel exposes a hierarchical view of all block devices and their attributes in the `sysfs` virtual filesystem.

### 3. `fdisk` Queries the Kernel

When you run a user-space utility like `fdisk` (or `lsblk`), it does not talk to the motherboard firmware or scan the hardware itself. Instead, it asks the Linux kernel for the information:

* `fdisk` typically reads from `/sys/block/` or uses system calls (`ioctl`) to query the kernel about the available block devices, their sizes, and partition tables.

### What role does the firmware play?

The firmware's job ends shortly after handing control to the bootloader and the kernel. While the UEFI/BIOS *does* provide initial hardware information to the operating system (via tables like ACPI), it is the **Linux kernel** that actively discovers, manages, and exposes the storage devices to user-space tools like `fdisk`.
