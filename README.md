# **randombullshit**

random collection of bash commands and other things

## **Arch Linux install**

---
### downloading arch iso

1. get version via

        $ wget -r --spider -l 1 http://mirror.digitalnova.at/archlinux/iso

2. and download corresponding version

        $ wget http://mirror.digitalnova.at/archlinux/iso/*version*/archlinux-*version*-x86_64.iso

### writing iso to usb drive

    $ dd if=path-to-iso of=path-to-usb-dev bs=4MB status=progress


### OPTIONAL: ssh

1. enable internet via *wifi-menu* or ethernet connection
2. set root passwd
3. start sshd.service and get ip with *ip a*

        $ ssh root@ip-addr

### partition drives

    $ lsblk
    $ gdisk /dev/drive
    # +512M ef00 partition for boot and the rest 8300 (linux partition)

### encrypting second partition

    $ modprobe dm-crypt
    $ cryptsetup -c aes-xts-plain64 -y -s 512 -h sha512 --i3000 luksFormat /dev/second-partition
    $ cryptsetup luksOpen /dev/second-partition *crypt-name*

### LVM

    $ pvcreate /dev/mapper/*crypt-name*
    $ vgcreate *vg-name* /dev/mapper/*crypt-name*
    $ lvcreate -L 80GB -n root *vg-name*
    $ lvcreate -L 16GB -n swap *vg-name*
    $ lvcreate -l 100%FREE -n home *vg-name*

### creating filesystems for volumes

    $ mkfs.fat -F32 -n UEFI /dev/first-partition
    $ mkfs.ext4 -L root /dev/mapper/*vg-name*-root
    $ mkfs.ext4 -L home /dev/mapper/*vg-name*-home
    $ mkswap -L swap /dev/mapper/*vg-name*-swap

### mounting partitions

    $ mkdir /mnt/{boot,home}
    $ mount /dev/mapper/*vg-name*-root /mnt
    $ mount /dev/mapper/*vg-name*-home /mnt/home
    $ mount /dev/second-partition /mnt/boot
    $ swapon /dev/mapper/*vg-name*-swap


### installing base system

    $ pacstrap /mnt base base-devel linux linux-firmware

    # additional packages
    gptfdisk dosfstools btrfs-progs e2fsprogs ntfs-3g lvm2 dhcpcd vim man-db man-pages texinfo networkmanager cpu_manufacturer-ucode

### generate fstab and chroot

    $ genfstab -Up /mnt >> /mnt/etc/fstab
    $ arch-chroot /mnt

### set timezone, locale and vconsole

    $ ln -sf /usr/share/zoneinfo/XX/XX /etc/localtime
    # uncomment locales local.gen file and run command
    $ vim /etc/locale.gen
    $ locale-gen
    $ echo KEYMAP=*keymap* > /etc/vconsole.conf

### network configuration

    $ echo *hostname* > /etc/hostname
    ---------------------------------------
    $ cat > /etc/hosts<< EOF
    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   *hostname*.localdomain   *hostname*
    EOF
    ---------------------------------------
    $ systemctl enable --now NetworkManager
    # test via ip a and ping

### mkinitcpio

    $ vim /etc/mkinitcpio.conf
    # add ext4 to MODULES
    # HOOKS=(base udev autodetect modconf block keyboard keymap encrypt lvm2 filesystems fsck)
    $ mkinitcpio -p linux

### systemdboot configuration

    $ bootctl install

    $ vim /boot/loader/loader.conf
    #
    default arch
    timeout 3
    editor 0

    $ blkid /dev/second-partition > /boot/loader/entries/arch.conf

    $ vim /boot/loader/entries/arch.conf
    #
    title   Arch Linux
    linux   /vmlinuz-linux
    initrd  /cpu_manufacturer-ucode.img
    initrd  /initramfs-linux.img
    options cryptdevice=UUID={UUID}:cryptlvm root=/dev/*vg-name*/root quiet rw

    $ cp /boot/loader/entries/arch.conf /boot/loader/entries/arch-fallback.conf
    # change title and initrd img to fallback

### set root passwd, exit chroot and reboot

    $ passwd
    $ exit
    $ umount /mnt/{boot,home,}
    $ swapoff -a
    $ reboot

## additional configuration once booting works

### adding user

    $ useradd -m -G wheel username
    $ passwd username
    # uncomment wheel in sudoers file
    $ EDITOR=/bin/vim visudo

### installing graphics drivers, xorg and minimal kde

    # check graphicscard and install drivers
    $ lspci -v | grep VGA
    $ pacman -Syu xf86-video-amdgpu/intel/nouveau mesa

    # install xorg server
    $ pacman -Syu xorg-server xorg-xinput xorg-xinit

    # install kde
    $ pacman -Syu plasma firefox dolphin terminator
    $ pacman -Syu --asdeps ffmpegthumbs kdegraphics-thumbnailers
    $ systemctl enable sddm
    $ reboot / systemctl start sddm

## some random packages, office, developing, etc.

    # nmap htop tmux samba wireshark
    # code dotnet-runtime dotnet-sdk nodejs npm python docker jre-openjdk jdk-openjk
    # libreoffice-fresh
