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
pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware dhcpcd iwd zram-generator nano intel-ucode git tmux usbutils unrar unzip p7zip unarchiver gvfs-mtp libmtp ntfs-3g android-udev mtpfs xdg-user-dirs xf86-video-intel vulkan-intel vulkan-icd-loader libva-intel-driver xorg-server xorg-xrdb xorg-xinit xorg-xrandr xorg-xev xorg-xdpyinfo xorg-xprop rustup neofetch htop btop fish pkgfile ffmpeg pulseaudio i3-wm rofi tlp acpid acpi_call upower discord feh sxhkd
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
```
<br>

```sh
nano /etc/hosts
```
```
127.0.0.1    localhost  
::1          localhost  
127.0.1.1    archbox.localdomain	  archbox
```
<br>

```sh
nano /etc/systemd/zram-generator.conf
```
```
[zram0]
zram-size = ram/8
compression-algorithm = zstd
swap-priority = 100

[zram1]
zram-size = ram/8
compression-algorithm = zstd
swap-priority = 100

[zram2]
zram-size = ram/8
compression-algorithm = zstd
swap-priority = 100

[zram3]
zram-size = ram/8
compression-algorithm = zstd
swap-priority = 100
```
<br>

```sh
nano /etc/pacman.conf
```
```
ParallelDownloads = 20
Color
ILoveCandy
VerbosePkgLists
[multilib]
Include = /etc/pacman.d/mirrorlist
```
<br>

```sh
nano /etc/mkinitcpio.conf
```
```
MODULES=(i915)
```
<br>

```sh
nano /etc/mkinitcpio.d/linux-zen.preset
```
```
PRESETS=('default')
```
<br>

```sh
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
```
%wheel ALL=(ALL:ALL) ALL
```

### Install systemd-boot
```sh
bootctl --path=/boot install
```
```sh
nano /boot/loader/entries/arch.conf
```
```
title Arch Linux
linux /vmlinuz-linux-zen
initrd /initramfs-linux-zen.img
initrd /intel-ucode.img
options root=/dev/sda2 rw
```
```sh
nano /boot/loader/loader.conf
```
```
default arch.conf
timeout 0
console-mode max
editor no
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

### Install paru (AUR)
```sh
git clone https://aur.archlinux.org/paru.git
cd paru
rustup default stable
makepkg -si
```

### Install and enable networkmanager w/ iwd
```sh
paru -S networkmanager-iwd
sudo systemctl disable iwd dhcpcd
sudo systemctl enable NetworkManager
reboot now
```

### Enable TLP for power management
```sh
sudo systemctl enable tlp acpid
```

### Install and configure ThinkFan
***not yet***

### Enable ALHP repos
Check for compatibility
```sh
/lib/ld-linux-x86-64.so.2 --help | grep supported # check for x86-64-vX
```
Install mirrors and keyring
```sh
paru -S -S alhp-keyring alhp-mirrorlist
```
Edit pacman.conf according to [ALHP guide](https://github.com/an0nfunc/ALHP#4-modify-etcpacmanconf)
```sh
sudo nano /etc/pacman.conf
```
Update system with new packages
```sh
sudo pacman -Syyu
```

### Set XORG to use hungarian layout
```sh
sudo nano /etc/X11/xorg.conf.d/00-keyboard.conf
```
```
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "hu"
EndSection
```

### Change xdg-user-dirs to name our home directories differently
```sh
nano /etc/xdg/user-dirs.defaults
```

### Install our display manager
```sh
paru -S ly-git
sudo systemctl diable getty@tty2
sudo systemctl enable ly
```

### Update the .bashrc to execute fish
```sh
if [[ $(ps --no-header --pid=$PPID --format=cmd) != "fish" ]]
then
        exec fish
fi
```

### Configure fish
```sh
# in fish
sudo pkgfile --update
fish_update_completions
curl -L https://get.oh-my.fish | fish
omf install archlinux bang-bang cd sudope batman
```

### Replace GTK3 with GTK3 Classic
```sh
paru -S gtk3-classic
```

### Install everyday apps
```
paru -S visual-studio-code-bin discord spotify teams ungoogled-chromium-bin
```

### Set up our .xinitrc
```sh
nano ~/.xinitrc
```
```sh
#!/bin/sh
# whatever customization you want
exec i3
```
<br>

```sh
chmod +x .xinitrc
```

## After all this, you should be good to go!
## If you wish to deploy my rice it can be found at [nuh uh not yet](#)