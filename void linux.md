## Installation

Download the glibc live iso image from the website . For ex: `void-live-x86_64-20250202-base.iso`
Insert the pen drive  and `lsblk -e7`
Look for your USB device (e.g.,` /dev/sdb, /dev/sdc`). Do NOT use a partition like `/dev/sdb1` — use the whole device (`/dev/sdb`).
 Unmount the USB
  ~~~
  sudo umount /dev/sdb
  ~~~
Write the ISO to usb
~~~
sudo dd if=path/to/void-live-x86_64-20250202-base.iso of=/dev/sdX bs=4M status=progress oflag=sync
~~~
Eject the USB

I am assuming the computer is connected to the internet using wired ethernet.

Boot the computer using the USB
At the prompt, enter username as `root` and password  `voidlinux`
Try pinging `ping voidlinux.org` to ensure internet access is available

#### Partitioning 
Select the hard disk for partitioning 
(#) lsblk -e7 
(#) cfdisk  /dev/sda
(#) Choose GPT
New: 
	1G
New:
	4G
New:
	Enter twice to select rest of the space 
write: yes
Enter to quit

Check the partitions , We have boot partition, a swap partition and a main partition
(#) lsblk 
Let us create the file system
(#) mkfs.ext4 /dev/sda3
(#)mkfs.fat -F 32 /dev/sda1
(#) mkswap /dev/sda2

(#) mount /dev/sda3 /mnt
(#) mount --mkdir /dev/sda1 /mnt/boot/efi
(#) swapon /dev/sda2

Check again
(#) lsblk 

Now let's install a minimal Void Linux system into /mnt by downloading the core packages from the official repository
(#) xbps-install  -Sy  -R  https://repo-default.voidlinux.org/current  -r  /mnt  base-system 

(#) clear
(#) xgenfstab  -U  /mnt  >  /mnt/etc/fstab
(#) xchroot /mnt /bin/bash

(#xchroot /mnt ) clear

(#xchroot /mnt ) xbps-install  base-devel  vim  grub  efibootmgr
(#xchroot /mnt ) vim /etc/hostname
	zentemple
	:wq
(#xchroot /mnt ) vim /etc/default/libc-locales
uncomment  en_us. UTF-8 . UTF-8
	:wq
(#xchroot /mnt ) xbps-reconfigure  -f  glibc-locales
(#xchroot /mnt ) clear
Set the root password
(#xchroot /mnt ) passwd 
	bodhi
(#xchroot /mnt ) clear
Add a user and set his password
(#xchroot /mnt ) useradd  -mG wheel narayan
(#xchroot /mnt ) passwd narayan
	rising
(#xchroot /mnt ) clear
(#xchroot /mnt ) EDITOR=vim visudo
Uncomment the following line 
%wheel  ALL=(ALL:ALL) ALL
	:wq
(#xchroot /mnt )  xbps-install  -S  grub-x86_64-efi 
(#xchroot /mnt ) 
grub-install  --target=x86_64-efi   --efi-directory=/boot/efi  --bootloader-id="Void"

(#xchroot /mnt ) grub-mkconfig  -o  /boot/grub/grub.cfg
(#xchroot /mnt ) clear

(#xchroot /mnt )  su  narayan
(narayan@zentemple  / $) sudo xbps-install  -Su

Change to home directory and set alias
(narayan@zentemple  / $) cd 
(narayan@zentemple  ~ $)  vim  .bashrc
alias xi = 'sudo xbps-install -Su'

(narayan@zentemple  ~ $) source .bashrc

(narayan@zentemple  ~ $) xi 





