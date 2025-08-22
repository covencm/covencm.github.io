---
title: "bspwm"
author: coven_cm
date: 2024-06-05T19:04:24+05:30
categories: [os, linux ]
draft: true 
tags: ["linux","theme"]
---
**Warning:** These are my personal configurations. You may need to edit  these files to fit your own preferences and setup.


![](https://raw.githubusercontent.com/esriee/dots/refs/heads/main/i3.png)

## What is bspwm

 BSPWM is a tiling window manager that arranges windows in a non-overlapping pattern. It is very easy to configure. but bspwm doesn't handle any keyboard or pointer inputs a third party program (e.g. sxhkd) is needed in order to translate keyboard and pointer events to bspc invocations.

### How to install bspwm 

Start by installing bspwm using your package manager. on Debian-based systems like Ubuntu, you can use

> `sudo apt install bspwm sxhkd polybar alacritty picom`

Arch Linux
>`sudo pacman -Sy bspwm sxhkd polybar alacritty picom `

Void Linux
>`sudo xbps-install bspwm sxhkd polybar alacritty picom`

### Configuration Files

If you want to use my configuration, which includes bspwm and other configuration files. Navigate to your home directory and execute

> `git clone https://github.com/covencm/dots`


###  Moving Configuration Files
Once cloned, first go into the cloned repository and locate the bspwm, polybar, and other configuration files within the dotfiles repository. Then, move these files to a directory named  ~/.config/ .  

to move all the configuration files at once, execute this command:

> `mv ~/dots/* ~/.config/`

### Permissions 

Make these files executable

>`chmod +x /home/(your user name here)/.config/bspwm/bspwmrc`

>`chmod +x /home/(your user name)/.config/sxhkd/sxhkdrc`


### Starting Bspwm 
To start bspwm, you need to set it as your default window manager.

If you are using a Login Manager, you might need to select bspwm on your login screen.

If not,  create or edit the **xinitrc** file and include the following line

> `exec bspwm`



That’s it!
