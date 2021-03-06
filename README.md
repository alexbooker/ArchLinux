# WIP - ArchLinux minimum Installation
my minimum Arch Linux Installution

This time, I'll install ArchLinux inside of VirtualBox ;)

## Requirement - ArchLinux ISO
from [here](https://www.archlinux.org/download/)

## Requirement - VirtualBox
from [here](https://www.virtualbox.org/wiki/Downloads)

## Requirement - At Least 2GB memory to skip swap step
## Step 1 - Check Internet Connection
```sh
$ ping -c 3 www.google.com
```

## Step 2 - Partitioning
#### type [ef02] on type definition for BIOS partition
TODO: Add Screenshot
- /dev/sda1 - boot
- /dev/sda2 - /
- /dev/sda3 - BIOS

#### type [ef00] on type definition for EFI partition
TODO: Add Screenshot
- FreeSpace
- /dev/sda1 - boot
- /dev/sda2 - EFI
- /dev/sda3 - /

## Step 3 - Formatting
#### BIOS
```sh
$ mkfs.ext4 /dev/sda1
$ mkfs.ext4 /dev/sda2
$ mkfs.ext4 /dev/sda3
```

#### EFI
```sh
$ mkfs.vfat -F32 /dev/sda3
$ mkfs.ext4 /dev/sda1
$ mkfs.ext4 /dev/sda2
```

## Step 4 - Mounting
#### BIOS
```sh
$ mount /dev/sda2 /mnt
$ mkdir -p /mnt/boot
$ mount /dev/sda1 /mnt/boot
```

#### EFI
```sh
$ mount /dev/sda3 /mnt
$ mkdir -p /mnt/boot
$ mount /dev/sda1 /mnt/boot
$ mkdir -p /mnt/boot/efi
$ mount /dev/sda2 /mnt/boot/efi
```

## Step 5 - MirrorList
```sh
$ vim /etc/pacman.d/mirrorlist
```
#### Find your country, and move it to the top of the list

## Step 6 - Install Base
#### BIOS
```sh
$ pacstrap /mnt base base-devel vim
```

#### EFI
```sh
$ pacstrap /mnt base base-devel vim efibootmgr
```

## Step 7 - fstab
```sh
$ genfstab -U -p /mnt >> /mnt/etc/fstab
```

## Step 8 - chroot
```sh
$ arch-chroot /mnt
```

## Step 9 - Locale
```sh
$ vim /etc/locale.gen
```
#### uncomment [en_US.UTF-8 UTF-8] and your language

## Step 10 - locale-gen
```sh
$ locale-gen
```

## Step 11 - Language setting
```sh
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ export LANG=en_US.UTF-8
```

## Step 12 - TimeZone & Hardware Clock
```sh
$ ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
$ hwclock --systohc --utc
```

## Step 13 - Hostname
```sh
$ echo "arch" > /etc/hostname
```

## Step 14 - Password
```sh
$ passwd
```

## Step 15 - BootLoader
#### BIOS
```sh
$ pacman -S grub
$ grub-install --recheck /dev/sda
```

#### EFI
```sh
$ pacman -S grub
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --recheck
```

## Step 16 - Setup GRUB main config
```sh
$ grub-mkconfig -o /boot/grub/grub.cfg
```

## Step 17 - netctl
```sh
$ pacman -S wireless_tools wpa_supplicant wpa_actiond dialog
$ systemctl enable netctl-auto@wlp2s0
$ systemctl enable dhcpcd.service
```

## Step 18 - Add User
```sh
$ useradd -m -g wheel yhoshino11
$ passwd yhoshino11
$ pacman -S sudo
$ visudo
```
```sh
# Uncomment these two lines
Defaults env_keep += "HOME"
%wheel ALL=(ALL) ALL
```

## Step 19 - Unmount & Reboot
```sh
$ exit
$ umount -R /mnt
$ reboot
```
Arch Machine Setup is Done !
Let's take a coffee break

=== After Installing Arch ===
## Step 1 - zsh
```sh
$ pacman -S zsh
$ chsh
$ /bin/zsh
```

## Step 2 - xorg
```sh
$ pacman -S xorg-server xorg-server-utils xorg-xinit xorg-xclock xterm
```

## Step 3 - Slim (wallpaper)
```sh
$ pacman -S slim archlinux-themes-slim slim-themes
$ vim /etc/slim.conf
```
#### Edit & Uncomment like this
```sh
login_cmd exec /bin/zsh -l ~/.xinitrc %session
daemon yes
default_user yhoshino11
focus_password yes
current_theme archlinux-simplyblack
```

## Step 4 - enable Slim for Login Screen
```sh
$ systemctl enable slim.service
```

## Step 5 - Xfce
```sh
$ pacman -S xfce4 xfce4-goodies gamin
```

## Step 6 - Start X at Login
```sh
$ cp /etc/X11/xinit/xinitrc ~/.xinitrc
$ vim ~/.xinitrc
```

replace this
```sh
twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66+0+0 -name login
```

to this
```sh
exec startxfce4
```

copy to yhoshino11's home directory
```sh
$ cp ~/.xinitrc /home/yhoshino11/
```
## Step 7 - Login with yhoshino11
```sh
$ su yhoshino11
$ cd
$ chsh
$ /bin/zsh
```

## Step 8 - enable Mouse & Keyboard
```sh
$ pacman -S xorg-drivers
```

## Step 9 - enable Disk Mount
```sh
$ pacman -S gvfs
```

## Step 10 - Virtualbox Guest Additions
```sh
$ pacman -S virtualbox-guest-utils
$ modprobe -a vboxguest vboxsf vboxvideo # try to copy things from host to guest with the clipboard to see if it works
$ VBoxClient-all
$ systemctl enable vboxservice
```

in ~/.xinitrc, add this line before exec startxfce4
```sh
# Read More https://wiki.archlinux.org/index.php/VirtualBox#Launch_the_VirtualBox_guest_services
/usr/bin/VBoxClient-all
```

## Step 11 - Install Chromium
```sh
$ pacman -S chromium
```

## Step 12 - Yaourt
```sh
$ vim /etc/pacman.conf
```
```sh
[multilib]
Include = /etc/pacman.d/mirrorlist # Uncomment
```

add this on bottom of file
```sh
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch

[pnsft-pur]
SigLevel = Optional TrustAll
Server = http://downloads.sourceforge.net/project/pnsft-aur/pur/$arch
```

run these commands
```sh
$ pacman -S --refresh yaourt
```
