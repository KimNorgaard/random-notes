# ArchLinux Install
This will install Arch Linux the following way:

   - single OS (deletes existing)
   - EFI boot using systemd-boot
   - full disk encryption using LUKS and LVM
   - one root partiton (ext4)
   - 8GB swap

## Pre-Requisites

   1. Update BIOS/UEFI firmware
   2. Update other firmware (dock, external things)
   3. Disable Secure Boot

## Create Installatoin Media

   1. Download ISO from https://www.archlinux.org/download/
   2. Copy the ISO to installation media:
     ````
     dd bs=4M if=arclinux.iso of=<usb device> status=progress
     ````

## Boot Using the Installation Media

## Installation
We will use /dev/nvme0n1 as the target disk.

### Disk Partitioning

````
cgdisk /dev/nvme0n1
````

   1. Delete all partitions
   2. Create partition 1, size 1GiB, type efi (ef00), name "esp")
   3. Create partition 2, size 100%, type vg (8e00), name "lvm")

### Setup ESP

````
mkfs.vfat -F32 /dev/nvme0n1p1
````

### Disk Encryption

Find a suitable cipher:

````
cryptsetup benchmark
````

Choose between fast and secure.

Create LUKS device:

````
cryptsetup --ciper aes-xts-plain64 --key-size 512 --hash sha512 -y \
           --use-random luksFormat /dev/nvme0n1p2
````

Choose a good password. Put it somewhere safe.

Open LUKS device:

````
cryptsetup luksOpen /dev/nvme0n1p2 luks
````

### Setup LVM

````
pvcreate /dev/mapper/luks
vgcreate vg00 /dev/mapper/luks
lvcreate --size 8G vg00 --name swap
lvcreate -l +100%FREE vg00 --name root
````

### Create File Systems and Swap

Create root file system. Leave 1% or root:
````
mkfs.ext4 -m 1 /dev/mapper/vg00-root
````

Create swap:
````
mkswap /dev/mapper/vg00-swap
````

### Mount File Systems

Mount root file system:
````
mount -o discard /dev/mapper/vg00-root /mnt
````

Mount swap:
````
swapon /dev/mapper/vg00-swap
````

Mount ESP:
````
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
````

### Install Base Packages

A few notes:
   - sudo and zfs is needed later when creating a user
   - dialog is needed by wifi-menu
   - wpa_supplicant and iw are needed for wireless networking
   - vim is always needed :)
````
pacstrap /mnt base base-devel vim sudo zsh dialog wpa_supplicant iw
````

### Generate /etc/fstab on Target System
````
genfstab -pU /mnt >> /mnt/etc/fstab
````

### OS Configuration

The remaining things are done via a chroot:

````
arch-chroot /mnt /bin/bash
````

#### Set Time Zone

````
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Europe/Copenhagen /etc/localtime
````

### Set Hardware Clock Using UTC

````
hwclock --systohc --utc
````

### Set the Host Name

````
echo changeme > /etc/hostname
````

### Locale Settings

````
vim /etc/locale.gen
````

Uncomment `da_DK.UTF-8 UTF-8` and `en_US.UTF-8 UTF-8`.

````
locale-gen

echo "LANG=en_US.UTF-8" >> /etc/locale.conf
````

### Set root Password

````
passwd
````

### Configure Initial Ram Disk for use with LUKS and LVM

````
vim /etc/mkinitcpio.conf
````

In the HOOKS variable, add `encrypt` and `lvm2` *before* `filesystem`, like
this:

````
HOOKS="base udev autodetect modconf block encrypt lvm2 filesystems keyboard fsck"
````

### Update Initial Ram Disk

````
mkinitcpio -p linux
````

### Install and Configure Boot Manager

````
bootctl install

echo "title ArchLinux" > /boot/loader/entries/arch.conf
echo "linux /vmlinuz-linux" >> /boot/loader/entries/arch.conf
echo "initrd /initramfs-linux.img" >> /boot/loader/entries/arch.conf

UUID=$(blkid -o value -s UUID /dev/nvme0n1p2)

echo "options cryptdevice=UUID=$UUID:vg:allow-discards root=/dev/mapper/vg00-root quiet rw" >> /boot/loader/entries/arch.conf
````

### Setup Console Font for HiDPI Displays

````
echo "FONT=latarcyrheb-sun32" > /etc/vconsole.conf
````

### Finish Installation

````
exit
umount -R /mnt
swapoff -a
reboot
````

## Post Install

### Networking

#### Wireless
Use `wifi-menu` to setup wireless.

#### Wired

Find out what interface to use:

````
ip a
````

Then write a netctl config:

````
vim /etc/netcfl/<if-name>
````

Content:

````
Description='Ethernet Connection'
Interface=<if-name>
Connection=ethernet
IP=dhcp
````

Start networking:

````
netctl start <if-name>
````


### Add a User

````
useradd -m -s /bin/zsh <username>
passwd <username>
````

### Allow the User to sudo as root

````
visudo  
````

Uncomment `%wheel ALL=(ALL) ALL`.

Add user to wheel group:

````
usermod -a -G wheel <username>  
````

