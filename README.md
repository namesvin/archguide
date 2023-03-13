## My personal Arch Linux installation guide
### For advanced users
Installation is done in the Arch Live ISO, which can be downloaded [here](https://mirror.vincuska.wtf/archlinux/iso/latest/archlinux-2023.02.01-x86_64.iso)

## Stage 1 - Live ISO

### Load my keymap
```sh
loadkeys hu
```

### Check for internet connectivity
```sh
ping gnu.org -c 2
```

### Partition disks
| Partition | Type | Size | Mountpoint |
|-----------|------|------|------------|
| /dev/sda1 | EFI  | 512M | /mnt/boot  |
| /dev/sda2 | Root | Rest | /mnt       |
```sh
cfdisk -z /dev/sda
```

### Format disks
```sh
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2 # Used to use xfs, until I tried to shrink my home partition :')
```

### Mount disks
```sh
mount /dev/sda2 /mnt
mount /dev/sda1 /mnt/boot
```

### Make our swapfile
```sh
dd if=/dev/zero of=/mnt/swap bs=1G count=4 # 1/4 of your ram
mkswap -U clear /mnt/swap
chmod 0600 /mnt/swap
swapon /mnt/swap
```

### Generate the fstab
```sh
genfstab /mnt > /mnt/etc/fstab
```

### Modify pacman.conf
```sh
# Uncomment ParallelDownloads and change the value to >10
nano /etc/pacman.conf
```

### Run pacstrap to install our system
```sh
pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware dhcpcd iwd zram-generator nano intel-ucode git tmux usbutils unrar unzip p7zip unarchiver gvfs-mtp libmtp ntfs-3g android-udev mtpfs xdg-user-dirs xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop rustup neofetch htop btop fish pkgfile ffmpeg pulseaudio i3-wm rofi
```

### Chroot into our system
```sh
arch-chroot /mnt
```

### Configure the boring shit
```sh
ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime

hwclock --systohc

echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

locale > /etc/locale.conf

echo -e "KEYMAP=hu\nFONT=default8x16" > /etc/vconsole.conf

echo "archbox" > /etc/hostname

nano /etc/hosts
# 127.0.0.1    localhost  
# ::1          localhost  
# 127.0.1.1    archbox.localdomain	  archbox

nano /etc/systemd/zram-generator.conf
#[zram0]
#zram-size = ram/8
#compression-algorithm = zstd
#swap-priority = 100
#
#[zram1]
#zram-size = ram/8
#compression-algorithm = zstd
#swap-priority = 100
#
#[zram2]
#zram-size = ram/8
#compression-algorithm = zstd
#swap-priority = 100
#
#[zram3]
#zram-size = ram/8
#compression-algorithm = zstd
#swap-priority = 100

nano /etc/pacman.conf
#ParallelDownloads = 20
#Color
#ILoveCandy
#VerbosePkgLists
#[multilib]
#Include = /etc/pacman.d/mirrorlist

nano /etc/mkinitcpio.conf
#MODULES=(i915)

mkinitcpio -p linux-zen
```

### Create our user
```sh
useradd -m -g users -G wheel,storage,power,video,audio,rfkill,input nya
```

### Set passwords
```sh
passwd
passwd nya
```

### Edit sudoers
```sh
nano /etc/sudoers
```

### Install systemd-boot
```sh
bootctl --path=/boot install

nano /boot/loader/entries/arch.conf
#title Arch Linux
#linux /vmlinuz-linux-zen
#initrd /initramfs-linux-zen.img
#initrd /intel-ucode.img
#options root=/dev/sda2 rw

nano /boot/loader/loader.conf
#default arch.conf
#timeout 0
#console-mode max
#editor no
```

### Enable internet
```sh
systemctl enable dhcpcd iwd
```

### Exit chroot and reboot
```sh
exit # or Ctrl+D
reboot
```

## Stage 2 - In our system