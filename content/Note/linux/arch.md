# 陷阱

1.在双系统的情况下，启动windows，windows 的自动更新(涉及固件更新的)可能会重置UEFL启动项(会使得光影精灵9 UEFL的安全启动被重新打开，这会使得linux 系统被禁止启动)，并且会使得grub丢失gruk:cfg 的路径，表现为开机进入grub 界面，无法进入任何系统

解决方案:

s1
```bash
search--file /grub/grub.cfg --set=root #每个系统的grub配置文件位置不一样configfile /boot/grub/grub.cfg
```

s2
按F10(每个电脑不一样)，进入bios 关闭安全启动