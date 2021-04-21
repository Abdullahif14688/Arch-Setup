Arch Linux and Awesome WM Install Notes
===================
So this install covers a fresh Arch Install, dual booting Windows on UEFI partition. 

## Table of Contents
1. [Pre Install](#preInstall)
    1. [Network Configuration](#network)
    2. [Partitioning Disks](#partition)
    3. [Formating Disk Partitioning](#format)
    4. [Mount the File System](#mount)
    5. [OPTIONAL: Change mirror list](#mirror)
2. [Installing Arch](#install)
3. [Configure the System](#config)
    1. [Change Hostname](#hostname)
    2. [Set Keyboard Layout](#keyboard)
    3. [Generate fstab](#fstab)
    4. [Setup Root Password](#root)
    5. [Setup Timezone](#time)
    6. [Setup Locale](#locale)
    7. [Install Grub (Bootloader)](#grub)
    8. [Enable dhcpcd](#dhcpcd)
    9. [Reboot Into Installed Media](#reboot)
   10. [Making Windows Visable to Grub](#windows)
4. [Post Install](#postInstall)
    1. [Installing Sudo](#sudo)
    2. [Setup User](#user)
    3. [Setup Graphics](#graphic)
    4. [Instal Build Utilities](#utils)
    5. [Install Yay (Package Manager)](#yay)
    6. [Setup Audio with Pulseaudio](#sound)
    7. [Install Neofetch](#neofetch)
    8. [Installing xorg](#xorg)
    9. [Setup Awesome (Window Manager)](#awesome)
   10. [Installing ly Login Manager](#ly)
5. [Startup GUI](#startx)

# Verify signature (optional) <a name="VerifySig"></a>
This step verifies the image signature of an arch iso to catch malicious images. ***Do this step before install***. Make sure you download the .sig file along with the iso. Then run the command below in terminal to verify the iso image. If everything looks good, then burn the iso to a usb and get ready to install Arch!
```{r, engine='bash', count_lines}
gpg --keyserver-options auto-key-retrieve --verify archlinux-*.iso.sig
```

# Pre Installation <a name="preInstall"></a>

### 1. Network configuration <a name="network"></a>
If using Ethernet, you can skip to step edit this to include contents ***************

Most laptops usually come with an atheros 
Wi-Fi chipset included.
If that is the case then use the daemon iwd to connect to a network.
To get an interactive prompt type the command below to open the iwctl client program.
```{r, engine='bash', count_lines}
iwctl
```
To list your wifi device type the following commmand.
```{r, engine='bash', count_lines}
[iwd]# device list
```
This should give you the name of your wifi device. Something like `wlan0`.
To connect to a network type the following commands, replace `wlan0` with your device name.
Replace `SSID` with the name of the network you want to connect to.
```{r, engine='bash', count_lines}
[iwd]# station wlan0 scan
[iwd]# station wlan0 get-networks
[iwd]# station wlan0 connect SSID
```

#### Verifiy connection
ping google.com or 8.8.8.8 
```{r, engine='bash', count_lines}
ping 8.8.8.8
```
### 2. Partitioning Disks <a name="partition"></a>

### List disks
```{r, engine='bash', count_lines}
lsblk
```


#### Make Swap and Linux Filesystem
partition scheme should be the following 
| partitions                                |
| :----------------------------------------:|
| efi partition created by windows          |
| linux swap should be 2x size of memory    |
| linux filesystem remaining available space |
```{r, engine='bash', count_lines}
cfdisk
```
If installing arch on NVME drive find name of drive using `lsblk`. The drive name should be something like `nvme0n1`. Then run cfdisk on this drive and create partitions on there.
```{r, engine='bash', count_lines}
cfdisk /dev/nvme0n1
```

### 3. Formating disk partitions <a name="format"></a>
the sda numbers can be different.
```{r, engine='bash', count_lines}
mkfs.btrfs /dev/sda1 #linux filesystem
mkswap /dev/sda3     #linux swap
swapon /dev/sda3
```
For an NVME drive
```{r, engine='bash', count_lines}
mkfs.btrfs /dev/nvme0n1p7 #linux filesystem
mkswap /dev/nvme0n1p6     #linux swap
swapon /dev/nvme0n1p6
```

### 4. Mount the file System <a name="mount"></a>
For a non-Nvme disk
```{r, engine='bash', count_lines}
mount /dev/sda1 /mnt      #linux filesystem
mkdir /mnt/{boot,home}
mount /dev/sda2 /mnt/boot #this should be the EFI partition
```
For an NVME disk
```{r, engine='bash', count_lines}
mount /dev/nvme0n1p7 /mnt      #linux filesystem
mkdir /mnt/{boot,home}
mount /dev/nvme0n1p1 /mnt/boot #this should be the EFI partition
```

### 5. Change mirror list - can skip see below <a name="mirror"></a>

#### Top 3 mirrors
| Mirrors               |
| :--------------------:|
| mirrors.unixheads.org |
| mirror.neotuli.net    |
| mirror.rit.edu        |

However this step is largely useless. 
Once connected to internet `reflector` will update mirror list automaticallly

To change enter command:
```{r, engine='bash', count_lines}
vim /etc/pacman.d/mirrorlist
```

# Installation <a name="install"></a>
Going to install archlinux base, linux and tools such as vim and vi.
```{r, engine='bash', count_lines}
pacstrap /mnt base base-devel linux linux-firmware vim vi
```
Install netctl and dependent packages to access wifi-menu.
```{r, engine='bash', count_lines}
pacstrap /mnt netctl dialog wpa_supplicant
```
So after installing Arch, we need to configure the system and install our desktop enviroment.

# Configure the system<a name="config"></a>

### 1. Change hostname <a name="hostname"></a>
```{r, engine='bash', count_lines}
hostnamectl set-hostname archbox
```
or 

```{r, engine='bash', count_lines}
echo archbox >> /mnt/etc/hostname
```

### 2. SetKeyboard Layout <a name="keyboard"></a>
Keyboard is preset to US,so no change needed. 
However, if you want to know how to change to non-US keyboards
check out [Arch Wiki for Details.](https://wiki.archlinux.org/index.php/installation_guide#Set_the_keyboard_layout)

### 3. Generate fstab <a name="fstab"></a>
```{r, engine='bash', count_lines}
genfstab -U -p /mnt > /mnt/etc/fstab
```

### 4. Change to root and change root password<a name="root"></a>
Enter the machine using the `arch-chroot ` command and create a password.
```{r, engine='bash', count_lines}
arch-chroot /mnt /bin/bash
passwd root
```

### 5. Setting timezone<a name="time"></a>
This sets timezone assuming central time. Change according to your local timezone.
```{r, engine='bash', count_lines}
ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime
hwclock --systohc --utc
```

### 6. Setting locale <a name="locale"></a>
Uncomment en_US.UTF-8 UTF-8 and other needed localizations in /etc/locale.gen.
```{r, engine='bash', count_lines}
vim /etc/locale.gen
```
Then generate the localizations.
```{r, engine='bash', count_lines}
locale-gen
touch /etc/locale.conf                        # Create config locale file
echo "LANG=en_US.UTF-8" >> /etc/locale.conf   # Add our locale to config file
```

### 7. Installing Grub (Our Bootloader)<a name="grub"></a>
Make sure that your efi partition is mounted to /mnt/boot.
if not run the command `mount /dev/sda# /mnt/boot`  or `mount /dev/nvme0n1p# /mnt/boot` where # is your efi boot partition number.
```{r, engine='bash', count_lines}
arch-chroot /mnt
pacman -S grub efibootmgr #Installing grub and efibootmgr

mkdir /boot/EFI/
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB

grub-mkconfig -o /boot/grub/grub.cfg
```

### 8. Install and enabling dhcpcd<a name="dhcpcd"></a>  
install dhcpcd
```{r, engine='bash', count_lines}
pacman -S dhcpcd
```
***Do not enable dhcpcd if you plan on using wifi to connect to the internet.***
This is to make sure your ethernet works when you boot into the installed media.

```{r, engine='bash', count_lines}
systemctl enable dhcpcd
```

### 9. Reboot into installed media<a name="reboot"></a>
umount -R unmounts the installed media recursivly. So both /mnt/home /mnt/boot and /mnt
would be unmounted.

```{r, engine='bash', count_lines}
exit
umount -R /mnt
swapoff /dev/sda3
reboot
```
On reboot windows isn't present but don't worry it will after some configuration.
Once you reboot annd login, inorder to connect to wifi run the command 
```{r, engine='bash', count_lines}
wifi-menu -o
````


### 10. Making Windows visable to grub<a name="windows"></a>
In order to make windows visable in grub bootloader you need a package called
OS-Prober. Install:
```{r, engine='bash', count_lines}
pacman -S os-prober
```
Now we have to update grub.cfg and reboot to check if windows appear.
```{r, engine='bash', count_lines}
grub-mkconfig -o /boot/grub/grub.cfg
reboot
```

# Post Installation ==> Installing Awesome WM and Light Display Manager<a name="postInstall"></a>

### 1. Updating Pacman and installing sudo<a name="sudo"></a>
```{r, engine='bash', count_lines}
pacman -Syu
pacman -S sudo
```

### 2. Setup user<a name="user"></a>
```{r, engine='bash', count_lines}
groupadd sudo

useradd -m -g sudo -s /bin/bash abfarah
passwd abfarah
visudo # give user sudo privlages
```
when doing `visudo` make sure to uncomment the `%sudo` line.
To make zsh the default terminal for the new user and root.
Get the lists of shells
```{r, engine='bash', count_lines}
 chsh -l
```
Change root's shell to zsh.
```{r, engine='bash', count_lines}
 chsh -s /usr/bin/zsh
```
Then lists the user's current shell.
```{r, engine='bash', count_lines}
grep abfarah /etc/passwd
```
Change the neew user's shell to zsh using usermod.
```{r, engine='bash', count_lines}
usermod --shell /usr/bin/zsh abfarah
```
confirm the shell was changed.
```{r, engine='bash', count_lines}
grep abfarah /etc/passwd
```
Log out and login to new user; in my case abfarah.

### 3. Xorg utilities and video drivers<a name="graphic"></a>
xf86-video-vesa has the nvideo drivers for GTX graphic cards. IE GTX3080. 
Make sure that yours matches.
```{r, engine='bash', count_lines}
sudo pacman -S xorg xorg-server xorg-server xorg-xinit xorg-apps mesa
sudo pacman -S xf86-video-nouveau nvidia
```

### 4. Build utilities<a name="utils"></a>
```{r, engine='bash', count_lines}
sudo pacman -S git wget curl zsh 
pacman -S multilib-devel fakeroot jshon make pkg-config autoconf automake patch
```

### 5. Installing yay and all of it's dependencies<a name="yay"></a>
```{r, engine='bash', count_lines}
git clone https://aur.archlinux.org/yay.git
cd yay 
makepkg -si
cd ..
rm -rf yay
```

### 6. Pulseaudio and Alsa sound utilities <a name="sound"></a>
pacmixer is our frontend for managing our audio input/output.
```{r, engine='bash', count_lines}
yay -S alsa-lib alsa-utils alsa-oss alsa-plugins
yay -S pulseaudio
yay -S pacmixer
```

### 7. Installing neofetch <a name="neofetch"></a>
```{r, engine='bash',count_lines}
yay -S neofetch
```

### 8. Installing xorg<a name="xorg"></a>
```{r, engine='bash', count_lines}
yay -S xorg xterm xorg-twm xorg-xclock
```

### 9. Installing and setting up awesome<a name="awesome"></a>
The ttf installes are fonts for awesome.
when changing `~/.xinitrc` make it match the `.xinitrc` file on github.
```{r, engine='bash', count_lines}
yay -S awesome
yay -S ttf-droid ttf-dejavu ttf-liberation
git clone https://github.com/abfarah/dotfiles.git
cp ./dotfiles/arch/.xinitrc ~/.xinitrc
mkdir -p .config/awesome
cp /etc/xdg/awesome/rc.lua .config/awesome/
cp -r /usr/share/awesome/* .config/awesome/
sudo yay -S rxvt-unicode pcmanfm
```

## 10. Installing [ly](https://github.com/nullgemm/ly). as our login manager <a name="ly"></a>
```{r, engine='bash', count_lines}
yay -S pam xorg-xauth
git clone https://github.com/nullgemm/ly.git
cd ly
make github
make
sudo make install
sudo systemctl enable ly.service
sudo systemctl disable getty@tty2.service
```
To change color of login manager You should add this line:
```{r, engine='bash', count_lines}
ExecStartPre=/usr/bin/printf '%%b' '\e]P0{background-color}\e]P7{foreground-color}\ec'
```
On file `/lib/systemd/system/ly.service`, line 8 (under type=idle). Change background-color and foreground-color to the RGB code of the desired color
My colors
```{r, engine='bash', count_lines}
ExecStartPre=/usr/bin/printf '%%b' '\e]P00A1529\e]P7FFFFFF\ec'
```

# Start up your GUI to see the magic<a name="startx"></a>
```{r, engine='bash', count_lines}
startx
```

The next steps from here will be to change default terminal to termite.
Installing oh-my-zsh and configuring vim. 
Theming and adding extra coconut oil to the rest of our build.
