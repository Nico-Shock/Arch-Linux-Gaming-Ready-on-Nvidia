# Arch Linux Installation Guide

# Requirements

1. **PC/Laptop**  
2. **UEFI Firmware is only recommendet**  
3. **Disabled Secure Boot**  
4. **A bootable USB stick with an ISO image**  
   - You can test the Arch Linux installation in a virtual machine.  
   - Download the Arch Linux ISO [here](https://archlinux.org/download/).  
5. **Tools to create a bootable USB stick:**  
   - **Windows:** Use [Rufus](https://rufus.ie/de/).  
   - **macOS:** Use [Balena Etcher](https://www.balena.io/etcher/).  
   - **Linux:** Use [ISO Image Writer](https://flathub.org/apps/details/org.kde.isoimagewriter).  
   - **For all OSes:** Install [Ventoy](https://www.ventoy.net/en/index.html) and copy the ISO to the root of your USB drive.
  
## **Please read and understand everything shown carefully before starting.**


| Step | Action |
|------|--------|
| **1** | [Set Up Keyboard Layout](#set-up-keyboard-layout) |
| **2** | [Check if Booted to UEFI](#check-if-booted-to-uefi) |
| **3** | [Check Internet Connection](#check-internet-connection) |
| **4** | [Connect to Wifi (if used)](#connect-to-wifi-if-used) |
| **5** | [Get Disk Information](#get-disk-information) |
| **6** | [Partition the Disk](#partition-the-disk) |
| **7** | [Format the Partitions](#format-the-partitions) |
| **8** | [Mount the Partitions to Install Linux Base System](#mount-the-partitions-to-install-linux-base-system) |
| **9** | [Make downloads faster](#make-downloads-faster) |
| **10** | [Install the Linux Base System](#install-linux-base-system) |
| **11** | [Setup swapfile](#setup-swapfile) |
| **12** | [Access the Root System](#access-the-root-system) |
| **13** | [Install Requirements](#install-requirements) |
| **14** | [Configure Arch System](#configure-arch-system) |
| **15** | [Install Bootloader](#install-bootloader) |
| **16** | [Install a Desktop Environment](#install-a-desktop-environment) |
| **17** | [Install Recommended and Required Stuff](#install-recommended-and-required-stuff) |
| **18** | [Install NVIDIA Drivers (Required to change the NVIDIA driver later)](#install-nvidia-drivers-required-to-change-the-nvidia-driver-later) |
| **19** | [Reboot System](#reboot-system) |
| **20** | [Change Keyboard Layout](#change-keyboard-layout) |
| **21** | [Change Timezone](#change-timezone) |
| **22** | [Settings needed to Install the Open NVIDIA Drivers from CachyOS](#settings-needed-to-install-the-open-nvidia-drivers-from-cachyos) |
| **23** | [Install CachyOS Repos](#install-cachyos-repos) |
| **24** | [Install yay and paru](#install-yay-and-paru) |
| **25** | [Install Chaotic-AUR Repos](#install-chaotic-aur-repos) |
| **26** | [Install CachyOS Kernel Manager & Other Things](#install-cachyos-kernel-manager--other-things) |
| **27** | [Install Open NVIDIA Driver (Recommended)](#install-open-nvidia-driver-recommended) |
| **28** | [Change Bootloaderconfig](#change-bootloaderconfig) |
| **29** | [Patch Pacman](#patch-pacman) |
| **30** | [Make Your System More Stable](#make-your-system-more-stable) |
| **Optional:** | |
| **31** | [Example for Gnome Theming](#theming-for-gnome) |
| **32** | [Recommended for VMware](#recommended-for-vmware) |
| **33** | [Recommended for KVM](#recommended-for-kvm) |

## Set Up Keyboard Layout

```
localectl list-keymaps
```
List all available keyboard layouts.

```
loadkeys de
```
Load the selected keyboard layout (default is US).

## Check if Booted to UEFI

```
efivar -l
```
If lots of unknown text appears on your screen that you don’t understand, it means you booted into UEFI mode.

## Check Internet Connection

```
ping archlinux.org
```
This will ping a website and show an output in bytes to check if you’re connected to the internet.

## Connect to Wifi (if used)

```
iwctl
```
Use this tool to show wifi information and connect to it.

```
station device scan
```
Replace `device` with your wifi adapter name.

```
station device get-networks
```
Replace `device` again with your WiFi adapter name and list the available networks.

```
station device connect Wifi-Name
```
Replace `Wifi-Name` with your real wifi name and enter the password.

## Get Disk Information

```
lsblk
```
This will show all information about your installed disks. You can use it anytime after changing your disks to display the updated information.

## Partition the Disk

```
cfdisk /dev/nvme0n1
```
This will use the `cfdisk` tool for easier partitioning of your disk.

### Disk Partitioning:
- Select **gpt**.
- Create a new partition with 2048M size and set the type to **EFI System**. If you using BIOS mode (lagacy csm mode) you need to set this optio nto biod boot.
- Use the remaining available space to create a final partition, leaving the type as default.

## Format the Partitions

```
mkfs.fat -F32 /dev/nvme0n1p1
```
Format the EFI/Boot partition to FAT32.

```
mkfs.ext4 /dev/nvme0n1p2
```
Format your Linux filesystem partition to the Linux ext4 file system.

## Mount the Partitions to Install Linux Base System

```
mount /dev/nvme0n1p2 /mnt
```
Mount your Linux filesystem partition to `/mnt`.

```
mkdir /mnt/boot
```
Create a directory for your bootloader.

```
mount /dev/nvme0n1p1 /mnt/boot
```
Mount the EFI/Boot partition to `/boot`.

## Make downloads faster

```
nano /etc/pacman.conf
```
Enable parallel downloads by removing the `#` from the respective line. (If you are unsure about the config, leave it at 5.)

## Install Linux Base System

```
pacstrap -K /mnt base base-devel linux linux-firmware
```
Install the base Linux system to your root directory.

```
genfstab -U -p /mnt >> /mnt/etc/fstab
```
Generate an `fstab` file.

## Setup swapfile

```
fallocate -l 4G /mnt/swapfile
```
Set your size and directory of your swap file.

```
chmod 600 /mnt/swapfile
```
Make your swap file only accessible by root for security reasons.

```
mkswap /mnt/swapfile
```
Format the swap file to the swap file system.

```
swapon /mnt/swapfile
```
Enable the swap file system.

```
nano /mnt/etc/fstab
```

-> you need to add `/swapfile none swap defaults 0 0` into that config, you can name it with `# swapfile` on a line above it if you want.

## Access the Root System

```
arch-chroot /mnt
```
Switch to the shell of your installed system.

## Install Requirements

```
pacman -S networkmanager nano
```
Install `NetworkManager`, and `nano`.

```
systemctl enable NetworkManager
```
Enable the `NetworkManager` service.

## Configure Arch System

```
nano /etc/locale.gen
```
Find your language (e.g., `de_DE`), uncomment it, and save changes.

```
locale-gen
```
Apply the language settings to your system.

```
echo LANG=de_DE.UTF-8 >> /etc/locale.conf
```
Add your language configuration.

```
export LANG=de_DE.UTF-8
```

```
echo iusearchbtw >> /etc/hostname
```
Replace `iusearchbtw` with your preferred hostname.

```
ln -sf /usr/share/zoneinfo/
```
Navigate to your timezone directory (e.g., `Europe/Berlin`) and link it:

```
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
```

```
hwclock --systohc --localtime
```

```
systemctl enable fstrim.timer
```

```
nano /etc/pacman.conf
```
- Uncomment `[multilib]` (not `multilib-testing`) and save the changes. You need this repo to access the additional packages you require.
- Enable parallel downloads by uncommenting the respective line. (Change the parallel downloads to whatever your internet is capable of, the maximum is 20.)

```
pacman -Sy
```

```
passwd
```
Set a password for the root user.

```
useradd -m -G wheel -s /bin/bash <username>
```
Replace `<username>` with your desired username.

```
passwd <username>
```
Set a password for your new user.

```
nano /etc/sudoers
```
Uncomment `%wheel ALL=(ALL) ALL`. (or just search for `# %`)
**Optional:** Add `Defaults rootpw` to require the root password for sudo at the bottom of the file.

## Install Bootloader

```
exit
systemctl daemon-reload
arch-chroot /mnt
```

```
bootctl install
```
Installs the bootloader.

```
nano /boot/loader/entries/arch.conf
```
Name the config file whatever you want.  
Paste this into the config file:

```
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
```
Name the title whatever you want, so you can replace "Arch Linux" with the name you want in your bootloader menu.

```
echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/nvme0n1p3) rw" >> /boot/loader/entries/arch.conf
```
Make sure to specify the correct disk, such as `/dev/nvme0n1p3`, for your root or system partition.
This adds your disk to your bootloader to boot Linux.

## Install a Desktop Environment

```
pacman -S gnome-shell gnome-terminal gnome-control-center gnome-software gnome-menus gnome-shell-extensions gnome-system-monitor mutter gdm
```

```
systemctl enable gdm
```
Or you can use only `gnome` as a package to install everything Gnome has. The example here is for a minimalistic build to create your own system without having lots of bloatware.

**There are many different desktop environments like `KDE Plasma` or `Cinnamon`. You can also use window managers like DWM, Hyprland, or i3, but these are for very experienced and advanced users. Additionally, you need slightly different installation steps for a window manager setup.**

## Install Recommended and Required Stuff

### Required Stuff

```
pacman -S flatpak git wget gedit thermald dolphin gnome-tweaks
```
Only install `dolphin` if you using KDE Plasma. And only install `gnome-tweaks` on Gnome.

### Recommended Stuff

```
pacman -S ufw fzf python python-pip bluez blueman bluez-utils zram-generator fastfetch preload
```

```
systemctl enable bluetooth ufw preload
```
You can only install `preload` after the installation of the Chaotic-AUR-Repos.

```
sudo ufw default deny
```
This will load default deny ufw settings.

```
sudo ufw allow ssh
```
If you need ssh, you can enable it with this command.

## Install NVIDIA Drivers (Required to change the NVIDIA driver later)

```
pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
```

```
pacman -S linux-headers
```

```
nano /etc/mkinitcpio.conf
```
Add the following to `MODULES=()`: `nvidia nvidia_modeset nvidia_uvm nvidia_drm`

```
mkdir /etc/pacman.d/hooks
nano /etc/pacman.d/hooks/nvidia.hook
```
Add the following:

```
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia

[Action]
Depends=mkinitcpio
When=PostTransaction
Exec=/usr/bin/mkinitcpio -P
```
You need the configuration for the `mkinitcpio.conf` to load the right modules for the nvidia driver and the hook to automatically update, remove, or install the nvidia-dkms driver. So, when you change the driver later to the open nvidia driver, everything will be automated with that step to remove the driver correctly.

## Reboot System

```
exit
umount -R /mnt
reboot
```

## Change Keyboard Layout

- Rightclick on your desktop and go to settings.
- Then go to "Keyboard" and select "Add Keyboard Layout."
- Choose your desired layout and delete the other layouts.

## Change Timezone

```
sudo timedatectl set-timezone Europe/Berlin
```
Select your right Timezone.

## Settings needed to Install the Open Nvidia Drivers from CachyOS:

```
sudo nano /etc/mkinitcpio.conf
```
Remove from `MODULES=()`: `nvidia nvidia_modeset nvidia_uvm nvidia_drm`

## Install CachyOS Repos

```
wget https://mirror.cachyos.org/cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
cd ..
sudo rm -r cachyos-repo.tar.xz cachyos-repo
```

## Install yay and paru.

```
sudo pacman -S yay
```

```
yay -S paru
```

## Install Chaotic-AUR-Repos

```
sudo -i
```

```
pacman-key --recv-key 3056513887B78AEB --keyserver keyserver.ubuntu.com
pacman-key --lsign-key 3056513887B78AEB
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-keyring.pkg.tar.zst'
pacman -U 'https://cdn-mirror.chaotic.cx/chaotic-aur/chaotic-mirrorlist.pkg.tar.zst'
```

```
exit
```

```
sudo gedit /etc/pacman.conf
```
Cut all the `cachyos` repos (from line 77 to line 87) to the end of the config. Remove the last 5 lines and replace them with the `cachyos` repos.  
Then add:

```
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```
to the bottom, like the other ones.

```
sudo pacman -Sy
```

## Install CachyOS Kernel Manager & Other Things

```
sudo pacman -S cachyos-kernel-manager cachyos-gaming-meta cachyos-settings
```

### Launch the CachyOS Kernel Manager

Select 'Configure' and under 'Options,' select the 'RC - Release Candidate,' then click on 'Build Kernel.' (Only if you do not have the latest RC kernel `linux-cachyos-rc`, or if you prefer to use a stable kernel with lower performance, you should use the `linux-cachyos` kernel.)  
After that, execute the installation to install the kernel.

## Install Open Nvidia Driver (Recommended)

```
sudo pacman -S linux-cachyos-nvidia-open
```

## Change Bootloaderconfig

```
sudo nano /boot/loader/entries/arch.conf
```

```
linux /vmlinuz-linux-cachyos
initrd /initramfs-linux-cachyos.img
```
Change the lines in the config to match and boot into the new kernel you installed.  

## Patch Pacman

```
sudo pacman -S pacman
```

## Make Your System More Stable

```
sudo pacman -Syuu
```
**Not really recommended:** If you have problems or some annoying outputs every time you use the pacman command, you can try disabling the `core` repo in `/etc/pacman.conf` by commenting it out with a `#`.

```
wget https://mirror.cachyos.org/cachyos-repo.tar.xz
tar xvf cachyos-repo.tar.xz && cd cachyos-repo
sudo ./cachyos-repo.sh
cd ..
sudo rm -r cachyos-repo.tar.xz cachyos-repo
```
If you break something, this will reinstall the CachyOS repos and should fix all pacman errors.

## Theming for Gnome

### Extensions for Gnome:

- Extension List
- Application and KStatusNotifierItem Support
- Blur My Shell
- Just Perfection
- Dash to Dock
- Arc Menu
- Coverflow
- Impatience
- Gnome 4x UI Improvements
- Caffeine
- Tiling Shell
- Compact Topbar
- Magic Lamp Effect
- Open Bar
- Highlight Focus
- User Themes
- System Monitor

For OpenBar pre-config download [this](https://github.com/Nico-Shock/sdfgdfg/releases/download/download/Purple) file and import it to OpenBar.

```
flatpak install flathub com.mattjakeman.ExtensionManager
```
Installs Extensions Manager.

```
yay -S candy-icons-git
```

```
yay -S ttf-dejavu-sans-mono-powerline-git
```
Then in Ptyxis go to settings and select a theme of your choice.  
I selected the 11th line from the bottom, the slightly darker purple.  
Select `DejaVu Sans Mono Bold 12` for the font.

Then I like to go to the Gnome Tweaks and change the normal fonts to Cantarell Bold with a size of 10. I also like to enable maximize and minimize buttons and later change only the shell theme to the WhiteSur theme.

```
git clone https://github.com/vinceliuice/WhiteSur-gtk-theme.git
cd WhiteSur-gtk-theme
./install.sh -l
cd ..
sudo rm -r WhiteSur-gtk-theme
```

```
sudo pacman -S zsh
```

```
sudo nano ~/.zshrc
```
Look for a theme, such as the [xiong-chiamiov-plus](https://github.com/ohmyzsh/ohmyzsh/blob/master/themes/xiong-chiamiov-plus.zsh-theme) theme.

Alternatively, you can try the [bira](https://github.com/ohmyzsh/ohmyzsh/blob/master/themes/bira.zsh-theme) theme.

I mainly use the [powerlevel10k](https://github.com/romkatv/powerlevel10k) theme.

```
chsh -s $(which zsh)
```
Make zsh your default terminal.

## Recommended for VMware:

```
sudo pacman -S open-vm-tools xf86-video-vmware xf86-input-vmmouse
```

```
sudo systemctl enable vmtoolsd
sudo systemctl start vmtoolsd
```

## Recommended for KVM:

```
sudo pacman -S spice-vdagent
```

```
sudo systemctl enable spice-vdagentd
sudo systemctl start spice-vdagentd
```
### *Make sure the theming steps are only examples of how I would theme my Linux on Gnome. You can customize it infinitely.*
