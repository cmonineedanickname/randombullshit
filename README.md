# **randombullshit**

random collection of bash commands and other things

## **Arch Linux install**

---
### **downloading arch iso**

1. get version via

        $ wget -r --spider -l 1 http://mirror.digitalnova.at/archlinux/iso

2. and download corresponding version

        $ wget http://mirror.digitalnova.at/archlinux/iso/*version*/archlinux-*version*-x86_64.iso

### **writing iso to usb drive**

    $ dd if=path-to-iso of=path-to-usb-dev bs=4MB status=progress


### **OPTIONAL: ssh**

1. enable internet via *wifi-menu* or ethernet connection
2. set root passwd
3. start sshd.service and get ip with *ip a*

        $ ssh root@ip-addr

### **partition drives**

    $ lsblk
    $ gdisk /dev/drive

+512M ef00 partition for boot
and the rest with 8300 (linux partition)

### **encrypting second partition**

    $ modprobe dm-crypt
    $ cryptsetup -c aes-xts-plain64 -y -s 512 -h sha512 --i3000 luksFormat /dev/second-partition
    $ cryptsetup luksOpen /dev/second-partition *crypt-name*

### **LVM**

    $ pvcreate /dev/mapper/*crypt-name*
    $ vgcreate *vg-name* /dev/mapper/*crypt-name*
    $ lvcreate -L 80GB -n root *vg-name*
    $ lvcreate -L 16GB -n swap *vg-name*
    $ lvcreate -l 100%FREE -n home *vg-name*

### **creating filesystems for partitions**

    $ mkfs.fat -F32 -n UEFI /dev/first-partition
    $ mkfs.ext4 -L root /dev/mapper/*vg-name*-root
    $ mkfs.ext4 -L home /dev/mapper/*vg-name*-home
    $ mkswap -L swap /dev/mapper/*vg-name*-swap

### **mounting partitions**

    $ mkdir /mnt/{boot,home}
    $ mount /dev/mapper/*vg-name*-root /mnt
    $ mount /dev/mapper/*vg-name*-home /mnt/home
    $ mount /dev/second-partition /mnt/boot
    $ swapon /dev/mapper/*vg-name*-swap


### **installing base system**

    $ pacstrap /mnt base base-devel linux linux-firmware

    # additional packages
    gptfdisk dosfstools btrfs-progs e2fsprogs ntfs-3g lvm2 dhcpcd vim man-db man-pages texinfo networkmanager

### **fstab and chroot**

    $ genfstab -Up /mnt >> /mnt/etc/fstab
    $ arch-chroot /mnt
### **timezone and locale**

    $ timedatectl, localectl

### **network configuration**

    $ hostnamectl

### **