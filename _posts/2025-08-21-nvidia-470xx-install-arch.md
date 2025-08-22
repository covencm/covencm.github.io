---
title: "nvidia-470xx-install-arch"
author: coven_cm
categories: [os, linux, nvidia ]
draft: true 
tags: ["linux","nvidia"]
---



The NVIDIA 470xx driver series is a legacy branch designed for older NVIDIA graphics cards that are no longer supported by the latest driver releases. These drivers are essential for users with legacy GPUs who want to maintain proper graphics acceleration and compatibility with modern Linux distributions.
This driver branch supports GPUs from the GeForce 600 series through some 900 series cards, as well as corresponding Quadro and Tesla models. While these are legacy drivers, they still receive critical security updates and bug fixes from NVIDIA.

**This article is recommended for people who still use Kepler (NVE0) series, GeForce 400/500/600 series cards or Tesla (NV50/G80-90-GT2XX) cards.**

**If you are using a GPU that supports Maxwell or higher architecture, it will be much easier for you to install.**


## Prerequisites for installing and building packages

First, update your system and install the required packages and kernel headers. After that, you need to enable 32-bit support by uncommenting the multilib repository. You are also going to need an AUR helper.


```bash 

#!/bin/bash

sudo pacman -Syu --noconfirm

sudo pacman -S base-devel git vim --needed --noconfirm

KERNEL=$(uname -r | cut -d'-' -f2-)
if [[ $KERNEL == *"zen"* ]]; then
    sudo pacman -S linux-zen-headers --noconfirm
elif [[ $KERNEL == *"lts"* ]]; then
    sudo pacman -S linux-lts-headers --noconfirm
elif [[ $KERNEL == *"hardened"* ]]; then
    sudo pacman -S linux-hardened-headers --noconfirm
else
    sudo pacman -S linux-headers --noconfirm
fi

sudo sed -i '/\[multilib\]/,/Include = \/etc\/pacman.d\/mirrorlist/ s/^#//' /etc/pacman.conf

sudo pacman -Sy --noconfirm

git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si --noconfirm
cd ..
rm -rf yay

```

##  Installing the base driver and OpenGL packages

If you are unsure about your NVIDIA card's supported driver family,  [you can find it  here](https://nouveau.freedesktop.org/CodeNames.html)



| Driver name                                             | Kernel                      | Base driver            | OpenGL             | OpenGL (multilib)        |
| ------------------------------------------------------- | --------------------------- | ---------------------- | ------------------ | ------------------------ |
| Turing (NV160/TUXXX) and newer                          | linux                       | nvidia-open            | nvidia-utils       | lib32-nvidia-utils       |
| Turing (NV160/TUXXX) and newer                          | any other kernel            | nvidia-open-dkms       | nvidia-utils       | lib32-nvidia-utils       |
| Maxwell (NV110) series up to Ada Lovelace (NV190/ADXXX) | linux                       | nvidia                 | nvidia-utils       | lib32-nvidia-utils       |
| Maxwell (NV110) series up to Ada Lovelace (NV190/ADXXX) | linux-lts                   | nvidia                 | nvidia-utils       | lib32-nvidia-utils       |
| Maxwell (NV110) series up to Ada Lovelace (NV190/ADXXX) | any other kernel            | nvidia-dkms            | nvidia-utils       | lib32-nvidia-utils       |
| Kepler (NVE0) series                                    | any                         | nvidia-470xx-dkms      | nvidia-470xx-utils | lib32-nvidia-470xx-utils |
| GeForce 400/500/600 series cards [NVCx and NVDx]        | any                         | nvidia-390xx-dkms      | nvidia-390xx-utils | lib32-nvidia-390xx-utils |
| Tesla (NV50/G80-90-GT2XX)                               | any                         | nvidia-340xx-dkms      | nvidia-340xx-utils | lib32-nvidia-340xx-utils |


```bash 

# Then install the correct drivers acording to the chart.  

# Example :
             yay -S nvidia-470xx-dkms nvidia-470xx-utils lib32-nvidia-470xx-utils

```

## Setting the Kernel Parameter and Add Early Loading of NVIDIA  Modules

If you use GRUB as the bootloader run, 

```bash 

sudo vim /etc/default/grub

Find the line that contains "GRUB_CMDLINE_LINUX_DEFAULT" and add these two parameters: nvidia-drm.modeset=1 nvidia-drm.fbdev=1

# Example 
    
    GRUB_CMDLINE_LINUX_DEFAULT="quiet splash nvidia-drm.modeset=1 nvidia-drm.fbdev=1"

# Update grub config 

    sudo grub-mkconfig -o /boot/grub/grub.cfg

```
 
 if you don't use GRUB as the bootloader, please check the documentation of your bootloader to find out how to add kernel parameters. [LILO](https://wiki.gentoo.org/wiki/LILO) [systemd-boot](https://wiki.archlinux.org/title/Kernel_parameters#systemd-boot)  [Limine](https://wiki.archlinux.org/title/Kernel_parameters#Limine)


## Enable Early Loading of NVIDIA Modules

```zsh

sudo vim /etc/mkinitcpio.conf

# Find the line that says  " HOOKS=() " and locate the word " kms "  in the parentheses and remove it. 
# after that Find the line that says MODULES=() 
# Modify the line to:
                     MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
                     
# Rebuild the initramfs with
                     sudo mkinitcpio -P
```

## pacman.hook

Use a pacman hook to auto-update initramfs after NVIDIA upgrades:

```zsh
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package

# ** IMPORTANT  ** Change nvidia to your specific driver package, e.g., nvidia-470xx-dkms

# e.g:  Modify Target=nvidia to Target=nvidia-470xx-dkms

#Target=nvidia 
Target=nvidia-470xx-dkms

Target=nvidia-open
Target=nvidia-lts
# If running a different kernel, modify below to match
Target=linux

[Action]
Description=Updating NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'

```
After that you need to Move the file to **/etc/pacman.d/hooks/** If the hooks folder doesn't already exist, you may need to create a new one.


## That's it 

Now you can reboot and enjoy! 

In case you face any issues related to NVIDIA graphics, you can check the [ArchWiki nvidia troubleshooting guide](https://wiki.archlinux.org/title/NVIDIA/Troubleshooting) .