<!--
# Installing Arch Linux on UEFI machines

Download the ISO from archlinux.org

Boot from the ISO

setfont ter-128n

dhcpcd

reflector --latest 5 --sort rate --save /etc/pacman.d/mirrorlist

pacman -Syy

ntpd -qg

Enable multilib

Install yay, fish, chromium and code
-->
