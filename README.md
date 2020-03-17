Arch Linux and Awesome WM Install Notes
===================
So this install covers a fresh Arch Install. This is installed using BIOS 
Firmware instead of UEFI firmware.

# Base Installation

### Network configuration
If using Ethernet, you can skip this step.

Most laptops usually come with an atheros 
Wi-Fi chipset included. If that’s the case
use wifi-menu then
```{r, engine='bash', count_lines}
wifi-menu
```

ping google.com or 8.8.8.8 to verify connection
```{r, engine='bash', count_lines}
ping 8.8.8.8
```

### Verify signature
This step verifies the image signature of arch iso to catch malicious images.
```{r, engine='bash', count_lines}
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```

### Update the system clock
Make sure the system clock is accurate with the following command
```{r, engine='bash', count_lines}
timedatectl set-ntp true
```

## Partitioning disks
#### Make Swap and Linux Filesystem
Run command below to partition disks
```{r, engine='bash', count_lines}
cfdisk
```
Create a sda1 partition and make it bootable<br/>
Create a sda2 partition for swap<br/>
Create a sda3 partition with rest of space for home directory

## Formatting disk partitions
For this step you want to format the partitions you just created.<br/>
For btrfs filesystem run:
```{r, engine='bash', count_lines}
mkfs.btrfs /dev/sda1
mkfs.btrfs /dev/sda3
```
For a ext4 filesystem run:
```{r, engine='bash', count_lines}
mkfs.ext4 /dev/sda1
mkfs.btrfs /dev/sda3
```
Next you want to make a swap partition and activate it.
```{r, engine='bash', count_lines}
mkswap /dev/sda2
swapon /dev/sda2
```

## Mount the file System
Mount the boot partition, create a home directory and mount it to sda3.
```{r, engine='bash', count_lines}
mount /dev/sda1 /mnt
mkdir /mnt/home
mount /dev/sda3 /mnt/home
```

## Change mirror list

#### Top 3 mirrors
| Mirrors               |
| :--------------------:|
| mirrors.unixheads.org |
| mirror.neotuli.net    |
| mirror.rit.edu        |

To change enter command:
```{r, engine='bash', count_lines}
vim /etc/pacman.d/mirrorlist
```

## Install arch Linux base system, Linux Kernel, and firmware.
```{r, engine='bash', count_lines}
pacstrap /mnt base linux linux-firmware
```
So after this the Arch is installed and just needs to be
configured for grub boot loader. 

# Configure the system

## change hostname
```{r, engine='bash', count_lines}
hostnamectl set-hostname archbox
```
or 

```{r, engine='bash', count_lines}
echo archbox >> /mnt/etc/hostname
```

## Set Keyboard Layout
Keyboard is preset to US so no change needed. However, 
if you want to know how to change to non-US keyboards
check out [Arch Wiki for Details.](https://wiki.archlinux.org/index.php/installation_guide#Set_the_keyboard_layout)

## generate fstab
```{r, engine='bash', count_lines}
genfstab -U -p /mnt > /mnt/etc/fstab
```

## Change root to new install and create root password
```{r, engine='bash', count_lines}
arch-chroot /mnt
passwd root
```

## Install Vim and man
```{r, engine='bash', count_lines}
pacman -S vim man
```

## Setting locale and time zone
Set system time zone to Central US time. Adjust this if in different time zone.
```{r, engine='bash', count_lines}
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc
```

Uncomment en_US.UTF-8 UTF-8 and other needed localizations in /etc/locale.gen, and generate them.
```{r, engine='bash', count_lines}
vim /etc/locale.gen
locale-gen
```
Create the local.conf file and set the Lang to en_US.UTF-8
```{r, engine='bash', count_lines}
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

## Network Configuration
Add the following to /etc/hosts
```{r, engine='bash', count_lines}
127.0.0.1   localhost
::1         localhost
127.0.1.1   archbox.localdomain	archbox
```

## Wifi-menu and dhcpd configuration.
Install dhcp and enable the dhcpcd daemon.
```{r, engine='bash', count_lines}
pacman -S dhcpcd
systemctl enable dhcpcd
```

To enable wifi-menu you need to install following packages in new install:
```{r, engine='bash', count_lines}
pacman -S netctl dialog wpa_supplicant
```

## setup user
To add user abdul and assign a password run command below:
```{r, engine='bash', count_lines}
useradd -m abfarah
passwd abfarah
```
install sudo package
```{r, engine='bash', count_lines}
pacman -S sudo
```
To add user abdul to wheel and other groups run command below:
```{r, engine='bash', count_lines}
usermod -aG wheel,audio,video,optical,storage,users abdul
```
Run `groups abdul` to verify user was added to groups.

## Give user sudo rights
Run `visudo` to open sudoers file and uncomment the line `#%wheel All=(All) All`.
This allows any users in wheel group to execute any command.

## Installing Grub
Download Grub package
```{r, engine='bash', count_lines}
pacman -S grub
```
Install and config grub bootloader
```{r, engine='bash', count_lines}
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## Reboot into installed media
After successfully installing and configuring grub, you can unmount and reboot into the fresh arch install.<br/>
`umount -R` unmounts the installed media recursively. So both /mnt/home /mnt/boot and /mnt
would be unmounted.

```{r, engine='bash', count_lines}
exit
umount -R /mnt
swapoff /dev/sda2
reboot
```
On reboot windows isn't present but don't worry it will after some configuration.

## Making Windows visible to grub
In order to make windows visible in grub bootloader you need a package called
OS-Prober. Install:
```{r, engine='bash', count_lines}
pacman -S os-prober
```
Now we have to update grub.cfg and reboot to check if windows appear.
```{r, engine='bash', count_lines}
grub-mkconfig -o /boot/grub/grub.cfg
reboot
```
# Post Installation ==> Installing Awesome WM and lightdm Login Manager

## Updating Pacman
```{r, engine='bash', count_lines}
pacman -Syu
```

## Enable 32-bit compatibility
```{r, engine='bash', count_lines}
vim /etc/pacman.conf # enable multilib
pacman -Syu
```

## Xorg installation
```{r, engine='bash', count_lines}
sudo pacman -S xorg xorg-xinit xf86-video-vesa xterm xorg-twm xorg-xclock
```

## Alsa sound utilities
```{r, engine='bash', count_lines}
sudo pacman -S alsa-lib alsa-utils alsa-oss alsa-plugins
sudo pacman -S git wget yajl
```

## build utilities
```{r, engine='bash', count_lines}
pacman -S multilib-devel fakeroot jshon make pkg-config autoconf automake patch which
```

## Install yay package manager
Install yay dependencies
```{r, engine='bash', count_lines}
sudo pacman -S binutils go gcc
```
Clone and install yay
```{r, engine='bash', count_lines}
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
Remove yay folder after install
```{r, engine='bash', count_lines}
rm -rf yay/
```

## Make WI-FI connect automatically
To list interface names type command below then tab and look for something starting with wlp.
```{r, engine='bash', count_lines}
ip link show dev [tab]
```
Enable WI-FI service using systemctl<br/>
(Assuming 'wlp3s0' is your interface name)
```{r, engine='bash', count_lines}
sudo systemctl enable netctl-auto@wlp3s0.service
```
To auto connect on start up
```{r, engine='bash', count_lines}
sudo netctl enable wlp3s0-WifiNameHere
```

## Change shell to zsh
Install zsh
```{r, engine='bash',count_lines}
yay -S zsh
```
Find zsh location
```{r, engine='bash',count_lines}
which zsh
```
Configure zsh: type following command and follow prompts.
```{r, engine='bash',count_lines}
zsh
```

## Installing archey3
```{r, engine='bash',count_lines}
yay -S archey3
```

## Installing and setting up awesome WM
```{r, engine='bash', count_lines}
yay -S awesome lightdm ttf-dejavu
```
Create directory and copy over config files for awesome WM
```{r, engine='bash', count_lines}
mkdir -p ~/.config/awesome
cp /etc/xdg/awesome/rc.lua ~/.config/awesome/
```

## Installing and configuring lightDM login manager
Install lightDM
```{r, engine='bash', count_lines}
yay -S lightdm
```
Configure lightdm. Open lightdm.conf file `sudo vim /etc/lightdm/lightdm.conf`.

### For autologin follow these steps
In the section labeled [Seat:*] find the following lines.
```{r, engine='bash', count_lines}
#autologin-user=
#autologin-session=
```

Uncomment and add the correct argument like so.
```{r, engine='bash', count_lines}
autologin-user=<username>
autologin-session=awesome
```
Enable the lightdm service.
```{r, engine='bash', count_lines}
systemctl enable lightdm.service
#Enter password when prompted
```

### For greeter and login screen follow these steps
Install a greeter of your choice. For the slick greeter run command below.
```{r, engine='bash', count_lines}
yay install lightdm-slick-greeter
```
Update the lightdm conf file to greeter. Uncomment the lightdm greeter and set to `lightdm-slick-greeter`.
```{r, engine='bash', count_lines}
sudo vim /etc/lightdm/lightdm.conf

```

## Install termite and set as default terminal
Install termite
```{r, engine='bash', count_lines}
yay -S termite
```
Set as default terminal in awesome lua file. And set terminal value to `termite`.
```{r, engine='bash', count_lines}
vim ~/.config/awesome/rc.lua
```

