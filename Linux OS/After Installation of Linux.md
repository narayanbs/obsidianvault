Installing microcode is a critical step for your 4th-gen i5 to ensure stability and security against CPU-level bugs (like Spectre or Meltdown). Since your processor is Intel, you'll be using the `intel-ucode` package.

The process differs significantly between the two because of how they handle the boot process.

---

## 1. Arch Linux

Arch makes this relatively straightforward, but you must manually tell your bootloader to load the microcode file **before** the main system image.

### Step A: Install the package

Bash

```
sudo pacman -S intel-ucode
```

### Step B: Update your Bootloader

The microcode needs to be loaded by the bootloader. Most Arch users use **GRUB**. If you are using GRUB, run the following command to automatically detect the microcode and update the configuration:

Bash

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

_Note: If you use `systemd-boot`, you must manually add `initrd /intel-ucode.img` as the first initrd line in your entry config file._

---

## 2. Void Linux

Void handles microcode via its package manager and a trigger system that rebuilds your "initramfs" (initial RAM filesystem).

### Step A: Enable the Non-free Repository

Microcode is proprietary, so it resides in the non-free repo.

Bash

```
sudo xbps-install -S void-repo-nonfree
```

### Step B: Install the package

Bash

```
sudo xbps-install -S intel-ucode
```

### Step C: Update the Initramfs

Void uses `dracut` by default to generate boot images. When you install `intel-ucode`, a hook usually triggers this automatically, but to be absolutely sure your boot image includes the new microcode, run:

Bash

```
sudo dracut --force
```

---

## How to Verify the Installation

After you have followed the steps for your chosen distro and **rebooted** your laptop, you can verify that the microcode was loaded successfully by checking the kernel logs.

Run this command in your terminal:

Bash

```
journalctl -k | grep microcode
```

_(On Void, if you don't have journalctl, use `dmesg | grep microcode`)_

**What to look for:**

You should see a message similar to this:

`[ 0.000000] microcode: sig=0x306c3, pf=0x2, revision=0x28, date=2019-11-12`

`[ 0.000000] microcode: Microcode Update Driver: v2.2.`

If the "revision" number in the second line is higher than the first, the update was successful.

---

### A Quick Warning for Arch Users

If you forget to run `grub-mkconfig` after installing the package, the microcode will sit on your disk but will **not** be loaded into the CPU at boot. This is a common "gotcha" for new Arch users.

Are you planning on using a specific bootloader, or are you sticking with the default GRUB?