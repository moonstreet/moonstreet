---
title: "A minimal Arch Linux installation with Gnome. My notes."
date: 2021-02-28T08:39:40+01:00
draft: false
showShare: false
codeMaxLines: 50
categories:
  - Technology 
tags:
  - linux
---

Arch Linux is extremely well documented so I highly recommend to read the [Arch installation guide](https://wiki.archlinux.org/index.php/Installation_guide).
This guide installs the Gnome desktop environment, but one can easily swap Gnome for i3 or another desktop environment, or none at all. )
This is the procedure I used for installing Arch on a Thinkpad t460s, t490s and a Dell Precision 5550.


## Partitioning

Checkout the current partition scheme and the name of the harddrive(s)

```shell
fdisk -l
```

Determine how you want to partition the disk. I do not use anything fancy (yet).

* a EFI boot partition and an ext4 root partition
* an encrypted root

| device       | size     | purpose     |
| :------------- | :----------: | -----------: |
|  /dev/nvme0n1p1 | 500 MB - 1 GB  | boot    |
| /dev/nvme0n1p2   | remainder | encrypted root| |

Let's go ahead and delete the existing partitions and create new partitions.

```shell
fdisk /dev/nvme01
fdisk d # delete until no partitions are left
fdisk n # boot partition, type +512M for size
fdisk n # for root partition, remainder of disk 
fdisk t L 1 # set to EFI
fdisk p # check
fdisk w # write
```

## Encrypt and mount

To encrypt the root partion with Luks:

```shell
cryptsetup -y -v luksFormat /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 cryptroot 
```
Source: https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LUKS_on_a_partition

Set filesystem to ext4 and mount it:

```shell
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
```

Make filesystem for boot and mount

```shell
mkfs.fat -F32 /dev/vme0n1p1
mkdir /mnt/boot
mount /dev/vme0n1p1 /mnt/boot
```


## Bootstrap

```sh
pacstrap /mnt vim sudo grub efibootmgr linux linux-lts base base-devel dhcpcd linux-firmware
```

Create fstab

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

Now chroot into the newly mounted root:

```shell
arch-chroot /mnt
```

Set time zone:

```shell
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime
hwclock --systohc

```

Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8 and other needed locales.

```shell
vim /etc/locale.gen
```

Generate the locales by running:

```shell
locale-gen
```
Create the locale.conf(5) file, and set the LANG variable accordingly:

```shell
/etc/locale.conf
LANG=en_US.UTF-8
```

Create the hostname file:

```shell
/etc/hostname
myhostname

/etc/hosts
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

## Grub

This step is the most exciting. We need to create a ramdisk to configure early userspace.
See here: https://en.wikipedia.org/wiki/Initial_ramdisk

* We need to make sure to add an encrypt hook before the filesystem is loaded
* We need to add the video driver so it starts before GDM (only for Gnome users)

Which videodriver to add? See here: https://wiki.archlinux.org/index.php/Kernel_mode_setting#Early_KMS_start


```sh
vim /etc/mkinitcpio.conf
MODULES=(i915)
HOOKS=(base udev autodetect keyboard keymap consolefont modconf block encrypt filesystems fsck)
```

Next generate the ramdisk:

```shell
mkinitcpio -P 
```

Now install Grub:

```shell
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
Edit the grub conf to point to the encrypted root. 

```shell
vim /etc/default/grub # --> cryptdevice=/dev/nvme0n1p2:cryptroot
```

Here is a screenshot from my grub config. I also changed the order as you can see.
![1](/grub.png)


Generate grub:

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

## Add user and desktop environment

```shell
passwd
useradd -mg users -G wheel,storage,power -s /bin/bash jacqueline
passwd jacqueline
pacman -S xorg xorg-server gnome zsh cmake git neofetch jq ansible
systemctl enable gdm.service
systemctl enable NetworkManager.service
systemctl enable dhcpcd
```

Now reboot into Gnome.

## Summary

### Add an extra encryption key

If you regret your disk encryption key, you can easily set another one:

```shell
sudo cryptsetup luksDump /dev/nvme0n1p2
sudo cryptsetup luksAddKey --key-slot 1 /dev/nvme0n1p2
```

### History


```sh
ln -sd /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime
hwclock --systohc
vim /etc/locale.gen
locale-gen
vim /etc/locale.conf
vim /etc/hostname
vim /etc/hosts
vim /etc/mkinitcpio.conf
mkinitcpio -P
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
vim /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
passwd
useradd -mg users -G wheel,storage,power -s /bin/bash jacqueline
passwd jacqueline
pacman -S xorg xorg-server gnome zsh cmake git neofetch jq ansible
systemctl enable gdm.service
systemctl enable NetworkManager.service
systemctl enable dhcpcd
```
