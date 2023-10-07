---
title: "archlinux 系统参数优化"
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
WLR_RENDERER=vulkan

QT_QPA_PLATFORM="wayland;xcb"
GDK_BACKEND=wayland,x11

#for firefox on wayland
MOZ_ENABLE_WAYLAND=1

#FOR QT THEME
#QT_QPA_PLATFORMTHEME=qt5ct
QT_STYLE_OVERRIDE=Breeze

EDITOR=nvim
VISUAL=nvim
#DESKTOP_SESSION=gnome
#DESKTOP_SESSION=plasma
```

## NVIDIA early KMS
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
root=PARTUUID=86db5b09-ce0d-4341-9773-fb2d39fddaad zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs radeon.dpm=1 nvidia_drm.modeset=1 amd_pstate=disable mitigations=off nowatchdog processor.ignore_ppc=1 nmi_watchdog=0 vm.dirty_writeback_centisecs=1500 quiet splash
```
如果你在使用auto-cpufreq的话需要把amd_pstate的值改为disable,否则使用passive

## 开机自动加载bbr模块
```
/etc/modules-load.d/bbr.conf
---------------------------------
tcp_bbr   #如果编译内核的时候把bbr编译进去就不用这个文件
```

## 设置默认tcp堵塞控制/qos算法/swapiness
```
/etc/sysctl.d/99-sysctl.conf
------------------------------------
kernel.unprivileged_bpf_disabled = 0
net.core.default_qdisc=cake
net.ipv4.tcp_congestion_control=bbr

vm.swappiness=180
vm.watermark_boost_factor=0
vm.watermark_scale_factor=125
vm.page-cluster=0

vm.dirty_writeback_centisecs=1500
vm.laptop_mode=5
```
## 启用与内存大小相同的zram来启用suspend
```
/etc/systemd/zram-generator.conf
-----------------------------------
[zram0]
zram-size = ram
compression-algorithm = zstd
fs-type = swap
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

## btrfs 优化
```
/etc/fstab
-----------------------------------
# /dev/nvme0n1p2
UUID=0eaaae30-9c36-4716-b7df-6e6bcf035464	/         	btrfs     	rw,relatime,ssd,space_cache=v2,noatime,commit=120,compress=zstd,discard=async,subvolid=256,subvol=/@	      0 0

# /dev/nvme0n1p1
UUID=4835-FFD2      	/efi     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	  0 2

# /dev/nvme0n1p2
UUID=0eaaae30-9c36-4716-b7df-6e6bcf035464	/.snapshots	btrfs     	rw,relatime,ssd,space_cache=v2,noatime,commit=120,compress=zstd,discard=async,subvolid=260,subvol=/@.snapshots	0 0

# /dev/nvme0n1p2
UUID=0eaaae30-9c36-4716-b7df-6e6bcf035464	/home     	btrfs     	rw,relatime,ssd,space_cache=v2,noatime,commit=120,compress=zstd,discard=async,subvolid=257,subvol=/@home	0 0

# /dev/nvme0n1p2
UUID=0eaaae30-9c36-4716-b7df-6e6bcf035464	/var/cache/pacman/pkg	btrfs     rw,relatime,ssd,space_cache=v2,noatime,commit=120,compress=zstd,discard=async,subvolid=259,subvol=/@pkg	0 0

# /dev/nvme0n1p2
UUID=0eaaae30-9c36-4716-b7df-6e6bcf035464	/var/log  	btrfs     	rw,relatime,ssd,space_cache=v2,noatime,commit=120,compress=zstd,discard=async,subvolid=258,subvol=/@log	0 0
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable fstrim.timer
```
