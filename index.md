---

# Arch Linux VM Installation Documentation

---

## 1. Introduction

This documentation outlines the process I followed to install and configure **Arch Linux** inside VM Workstation Pro 17.6.4 for CYB-3353-01. This document also serves as a reference for reproducing my setup, if needed, and includes comments on any difficulties encountered.

---

## 2. Virtual Machine Setup

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

## 3. Boot and Initial Configuration

When the Arch Linux screen pops up, select:

**Arch Linux install medium (x86_64 UEFI)**

### Console Keyboard Layout and Font

The default is US, so there's no need to change it.

### 3.1 Change Font to be larger and more readable
```
# stefont ter-132b
```

### Verification of Boot Mode

```
# cat /sys/firmware/efi/fw_platform_size
```

Looking for the return value of `64`, indicating that it booted in 64-bit x64 UEFI mode.

---

## 4. Network Configuration

Check network connectivity:

```
# ping -c 5 www.google.com
```

Should return 5 packets transmitted, 0% packet loss, indicating network connectivity.

Update system clock:

```
# timedatectl set-ntp true
```

---

## 5. Disk Partitioning

List block devices:

```
# lsblk
```

Partition the disk:

```
# fdisk /dev/sda
```

Create GPT table:

```
g
```

### EFI Partition

```
n
+1G
t
1
```

### Swap Partition

```
n
+4G
t
19
```

### Root Partition

```
n
(defaults)
t
23
```

Write changes:

```
w
```

---

## 6. Formatting Partitions

```
# mkfs.fat -F32 /dev/sda1
# mkswap /dev/sda2
# mkfs.ext4 /dev/sda3
```

---

## 7. Mounting the File System

```
# mount /dev/sda3 /mnt
# mount --mkdir /dev/sda1 /mnt/boot
# swapon /dev/sda2
```

---

## 8. Base System Installation

```
# pacstrap -K /mnt base linux linux-firmware nano vim sudo man reflector grub efibootmgr networkmanager dhcp dhcpcd wget git
```

---

## 9. System Configuration

### Fstab

```
# genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

```
# arch-chroot /mnt
```

### Time

```
# ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime 
# hwclock --systohc
```

### Localization

```
# nano /etc/locale.gen
```

Uncomment:

```
#en_US.UTF-8 UTF-8
```

Then:

```
# locale-gen
# echo "LANG=en_US.UTF-8" > /etc/locale.conf
# echo "KEYMAP=us" > /etc/vconsole.conf
```

---

## 10. Network Setup

Set hostname:

```
# echo atlan > /etc/hostname
```

Edit hosts:

```
# nano /etc/hosts
127.0.1.1 atlan
```

Enable Network Manager:

```
systemctl enable NetworkManager
pacman -S dhcpcd networkmanager resolvconf
systemctl enable dhcpcd
systemctl enable NetworkManager
systemctl enable systemd-resolved
```

---

## 11. User Setup

Set root password:

```
# passwd
atlan
```

Create users:

```
# useradd -m -G wheel -s /bin/bash atlan
# passwd atlan
# useradd -m -G wheel -s /bin/bash codi
# passwd codi
```

### Enable Sudo

```
# EDITOR=nano visudo
```

Uncomment:

```
#%wheel ALL=(ALL:ALL) ALL
```

---

## 12. Bootloader
Ensure that you are chroot into /mnt or this will not setup properly
```
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
# grub-mkconfig -o /boot/grub/grub.cfg
```

Exit and reboot:

```
# exit
# umount -R /mnt
# swapoff -a
# reboot
```

---

## 13. Post-Install Configuration

Login and update:

```
sudo pacman -Syu
```

Install KDE Plasma:

```
sudo pacman -Syu coreutils diffutils plasma-meta sddm dolphin kde-applications konsole xdg-utils
```

Enable SSH:

```
sudo pacman -S openssh
sudo systemctl enable sshd --now
```

---

## 14. Zsh Configuration

Install Zsh:

```
sudo pacman -S zsh zsh-completions zsh-autosuggestions zsh-syntax-highlighting
```

Enable SDDM:

```
sudo systemctl enable sddm.service
sudo systemctl set-default graphical.target
```

Set Zsh as the default:

```
sudo chsh -s /usr/bin/zsh
exec zsh
```

Zsh setup:

```
% autoload -Uz zsh-newuser-install
% zsh-newuser-install -f
```

Confirm:

```
% echo $SHELL
```

Should show `/usr/bin/zsh`.

---

## 15. Terminal Customization

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
% source ~/.zshrc
```

---

## 16. Boot into KDE

```
sudo systemctl start sddm
```

Log in and confirm the graphical interface.
You now have a functional Arch Linux installation with KDE and Zsh.

---

## 17. Firefox Installation

```
sudo pacman -Syu firefox
```

---

## 18. Difficulties

| **Issue**           | **Description**                                                 |
| ------------------- | --------------------------------------------------------------- |
| Performance Lag     | Severe lag caused slow input responses.  Got stuck in the partition type menu due to severe input lag. I have not found the cause or solution for this yet  |                        |
| Mount/Grub Issues   | Likely Caused by incorrect esp `/mnt` vs `/mnt/boot` handling due to chroot-ing in the incorrect order.      |
| Reflector Timeout   | Mirrorlist generation failed, likely due to network delays.             |

---


