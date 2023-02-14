---
title: "一些系统设置"
date: 2023-02-14T13:56:27+08:00
draft: false
tags: ["linux"]
---

# 一些系统设置

## 系统环境变量
```
#
# This file is parsed by pam_env module
#
# Syntax: simple "KEY=VAL" pairs on separate lines
#
#

##for fcitx5
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus

#hydrland
LIBVA_DRIVER_NAME=nvidia
XDG_SESSION_TYPE=wayland
GBM_BACKEND=nvidia-drm ##强制将使用 GBM 后端
__GLX_VENDOR_LIBRARY_NAME=nvidia ##强制将使用 GBM 后端
WLR_NO_HARDWARE_CURSORS=1

#enable nvidia vertical sync
#__GL_SYNC_TO_VBLANK=1

# set backend as vulkan
#WLR_RENDERER=vulkan

#for obs on wayland(to use pipewire screen capture)
QT_QPA_PLATFORM=wayland
#for firefox on wayland
MOZ_ENABLE_WAYLAND=1
#FOR QT THEME
QT_QPA_PLATFORMTHEME=qt5ct


#DESKTOP_SESSION=gnome
#DESKTOP_SESSION=plasma
```


## nvidia early kms
```
/etc/mkinitcpio.conf
-----------------------------------
MODULES=(btrfs amdgpu nvidia nvidia_modeset nvidia_uvm nvidia_drm)
BINARIES=(/usr/bin/btrfs)
FILES=()
HOOKS=(base udev autodetect keyboard keymap modconf block filesystems resume fsck)
```
`resume` 和下面的内核参数`resume=/dev/zram0`配合来启用suspend

```
/etc/nvidia.conf
---------------------------------
options nvidia-drm modeset=1
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```

## 内核参数
```
/boot/loader/entries/***.config  options root=...
or  
/boot/grub/grub.cfg   GRUB_CMDLINE_LINUX_DEFAULT=...  
----------------------------------
PARTUUID=86db5b09-ce0d-4341-9773-fb2d39fddaad zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs radeon.dpm=1 resume=/dev/zram0
```
关闭zswap,使用zram

## 开机自动加载bbr模块
```
/etc/modules-load.d/bbr.conf
---------------------------------
tcp_bbr   #如果编译内核的时候把bbr编译进去就不用这个文件
```

## 设置默认tcp堵塞控制/qos算法
```
/etc/sysctl.d/99-sysctl.conf
------------------------------------
net.core.default_qdisc=cake
net.ipv4.tcp_congestion_control=bbr2
```
## 启用与内存大小相同的zram来启用suspend
```
/etc/systemd/zram-generator.conf
-----------------------------------
[zram0]
zram-size = ram
compression-algorithm = zstd
```

## X11
用`sudo nvidia-xconfig`生成`/etc/X11/xorg.conf`  
```
a+n用户需要：
/etc/X11/xorg.conf.d/20-amdgpu.conf
----------------------------------
Section "Device"
	Identifier "AMD"
	Driver "amdgpu"
EndSection
```

## 休眠相关
如果n卡用户休眠睡死请检查是否启用**对应**的服务
`sudo systemctl enable nvidia-suspend.service`
