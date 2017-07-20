---
title: "Notes-HowToBackupTheWholeSystem"
category: HackING
layout: post
author: A1phaZer0
---
```sh
dd if=/dev/sdX of=/dev/sdY
```
sdX is source, sdY is destination.
This is a byte by byte clone, so the whole system is mirrored to new hdd. Because superblock is cloned also, the UUID of both hdd will be the same, no need to worry about GRUB or something.  
If you got a superblock error, try `Alt + SysRq + R E I S U B`(**R**eboot **E**ven **I**f **S**ystem **U**tterly **B**roken), after reboot, do dumpe2fs to check backup superblocks.
```sh
sudo dumpe2fs /dev/sdY | grep superblock
```
Use alternative superblock to restore filesystem, rest is pure magic.
```sh
sudo fsck -b superblock_id -y /dev/sdY
```
After all things done, use gparted or other tools to resize partitions if needed.
