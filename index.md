---

# Arch Linux VM Installation Documentation

---

## 1. Virtual Machine Setup

### Create a New VM in VMware Workstation (Typical Configuration Option)

| **Setting**                   | **Value**                           |
| ----------------------------- | ----------------------------------- |
| Guest Operating System        | Linux                               |
| Version                       | Other Linux 6.x kernel 64-bit       |
| VM Name                       | Arch-Linux                          |
| Number of Processors          | 7                                   |
| Number of Cores per Processor | 2                                   |
| Memory                        | 16GB RAM                            |
| Maximum Disk Size             | 40GB                                |
| Disk Type                     | Store virtual disk as a single file |
| Accelerate 3D Graphics        | Enabled                             |
| Graphics Memory               | 8GB                                 |

**Finish Setup**

* Set CD/DVD to detect the downloaded Arch ISO image file from [official download page](https://archlinux.org/download/).
* Once the VM is made in Workstation:
  Go to **VM Settings → Options → Advanced → Firmware Type → UEFI Mode**.

Now, power on the VM.

---

## 2. Boot and Initial Configuration

When the Arch Linux screen pops up, select:

**Arch Linux install medium (x86_64 UEFI)**

### Console Keyboard Layout and Font

The default is US, so there was no need to change it, but if needed:
```
Available layouts can be listed with:

# localectl list-keymaps

Can change the layout with:

# loadkeys  [layout]
```

### Change Font to be readable
```
# stefont ter-132b
```

### Verification of Boot Mode
Concatenate the UEFI bit
```
# cat /sys/firmware/efi/fw_platform_size
```

Looking for the return value of `64`, indicating that it booted in 64-bit x64 UEFI mode.

---

## 3. Network Configuration
Note: Because this was done in a VM using NAT, the network configuration was automatically configured as an Ethernet connection

Check network connectivity:
```
# ping -c 5 www.google.com
```

Should return 5 packets transmitted, 0% packet loss, indicating network connectivity.

Update system clock
```
# timedatectl set-ntp true
```

---

## 4. Disk Partitioning (UEFI with GPT layout on the Installation Guide)

List block devices:

```
# lsblk
```

Partition the disk (sda):

```
# fdisk /dev/sda
```

Create GPT table:

```
g
```

### EFI Partition (sda1)

```
n
default
default
+1G

t
1
```

### Swap Partition (sda2)

```
n
default
default
+4G

t
19
```

### Root Partition (sda3)

```
n
default
default
default (will assign the rest of the space to this partition)

t
23
```

Write changes:

```
w
```

---

## 5. Formatting Partitions

```
The first partition, the EFI system partition, needs the FAT32 file system
# mkfs.fat -F32 /dev/sda1

Initialize swap space, as the partition does not have a file system
# mkswap /dev/sda2

The third partition, the Linux root (x86-64), should have the  ext4 file system
# mkfs.ext4 /dev/sda3
```

---

## 6. Mounting the File System and Enabling the Swap Partition

```
(Pay attention to the order and /mnt vs /mnt/boot. This is likely the cause of the failure of my first attempt.)
# mount /dev/sda3 /mnt
# mount --mkdir /dev/sda1 /mnt/boot
# swapon /dev/sda2
```

---

## 7. Base System Installation

```
Recommended minimum new system installation from scratch.
# pacstrap -K /mnt base linux linux-firmware nano vim sudo man reflector grub efibootmgr networkmanager dhcp dhcpcd wget git resolvconf
```

---

## 8. System Configuration

### Fstab

```
Generate File System Table
# genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot


```
Changes the root directory for the current running process and its children
# arch-chroot /mnt
```

### Time Configuration

```
Symbolic link of the proper timezone to the localtime file
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
Set the system hardware clock
# hwclock --systohc
```

### Localization

```
# nano /etc/locale.gen
```

Uncomment by removing '#' in front of the line

```
#en_US.UTF-8 UTF-8
```

Then:

```
generate locales
# locale-gen
Set the default language to US.UTF-8
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
Set key map to us
# echo "KEYMAP=us" > /etc/vconsole.conf
```

---

## 9. Network Setup

Set hostname:

```
Set system name to "atlan"
# echo atlan > /etc/hostname
```

Edit hosts:

```
# nano /etc/hosts
Insert:
127.0.1.1 atlan
```

Enable Network Manager:

```
# systemctl enable NetworkManager
# systemctl enable dhcpcd
# systemctl enable systemd-resolved
```

---

## 10. User Setup

Set root password:

```
# passwd
atlan
```

Create users:

```
Create user atlan and add them to wheel group and default shell bash
# useradd -m -G wheel -s /bin/bash atlan
Set passwd for atlan
# passwd atlan
atlan
Create user codi and add them to wheel group and default shell bash
# useradd -m -G wheel -s /bin/bash codi
Set passwd for atlan
# passwd codi
codi
```

### Enable Sudo

```
# EDITOR=nano visudo
```

Uncomment by removing '#' in front of the line

```
#%wheel ALL=(ALL:ALL) ALL
```

---

## 11. Bootloader
Ensure that you are chrooted into /mnt, or this will not function properly and you will experience great pain
```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

Exit and reboot:

```
Exit chroot
# exit
Unmount partitions recursively under /mnt
# umount -R /mnt
Disable swap space
# swapoff -a
# reboot
```

---

## 12. Post-Install Configuration

Update Package Manager

```
#sudo pacman -Syu
```

Install KDE Plasma:

```
#sudo pacman -Syu coreutils diffutils plasma-meta sddm dolphin kde-applications konsole xdg-utils
```

Enable SDDM (KDE Graphical Display):

```
#sudo systemctl enable sddm.service
#sudo systemctl set-default graphical.target
```

Enable SSH:

```
#sudo pacman -S openssh
#sudo systemctl enable sshd --now
```

---

## 13. Zsh

Install Zsh:

```
#sudo pacman -S zsh zsh-completions zsh-autosuggestions zsh-syntax-highlighting
```

Set Zsh as the default:

```
Change the default terminal to zsh
#sudo chsh -s /usr/bin/zsh
#exec zsh
```

Zsh setup:

```
Load defaults for a new user
% autoload -Uz zsh-newuser-install
% zsh-newuser-install -f
```

Confirm:

```
% echo $SHELL
```

Should show `/usr/bin/zsh`.

---

## 14. Terminal Customization

Edit `~/.zshrc`:

```
% nano ~/.zshrc/
```

Add after “compinit”:

```
# Enable command completion
autoload -Uz compinit && compinit

# Enable color support
autoload -Uz colors && colors

# Color Prompt
setopt PROMPT_SUBST
PROMPT='%F{cyan}%n%f@%F{green}%m%f %F{blue}%~%f %# '

# Aliases
alias ls='ls --color=auto'
alias ll='ls -l --color=auto'
alias la='ls -la --color=auto'
alias grep='grep --color=auto'
alias diff='diff --color=auto'
alias mkdir='mkdir -pv'
alias ..='cd ..'
alias ...='cd ../../'
alias try='xdg-open'
alias ports='sudo netstat -tulnp'
alias mem='free -h'
alias cpu='lscpu | grep "Model name"'
alias wget='wget -c'
```

Save and reload:

```
%source ~/.zshrc
```

---

## 15. Boot into KDE
Start the Graphical Login Manager
```
%sudo systemctl start sddm
```

Log in and confirm the graphical interface.
You now have a functional Arch Linux installation with KDE and Zsh.

---


## 17. Difficulties/Questions

| **Issue/Question** | **Description/Solution** |
| ------------------- | ------------------------ |
| I: Performance Lag | Severe lag caused slow input responses. Got stuck in the partition type menu among other menus, or when editing using nano multiple times due to severe input lag, which also significantly slowed down every step of the process. I have not found the cause or solution for this yet. It has to do with the VM and my host machine itself; it has been reduced when in Konsole in KDE. |
| I: Mount/Grub Issues | Caused by incorrect `esp`, `/mnt` vs `/mnt/boot` handling due to chroot-ing in the incorrect order. Corrected by doing it in the correct order |
| I: Pacstrap failing | Pacstrap command failing due to attempting to install packages it doesn't have and typos. Corrected by retrying without typos/incorrect packages |
| I: Reflector Timeout | Mirrorlist generation failed, likely due to network delays. Corrected by trying again at a later time. |
| Q: How to Customize the terminal | Editing `~/.zshrc/` and using alias, PROMPT for coloring, and `autoload -Uz [options…]`. (I don’t know why I made it part blue — anything in the color blue is nearly unreadable for me.) |

---


