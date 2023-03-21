---
title: "Archlinux安装小记"
date: 2022-02-17T00:56:27+08:00
draft: false
tags: ["linux"]
ShowToc: true
---

# 创建安装介质
工具：rufus  
环境：windows  
去[archlinux](https://archlinux.org/)官网安装即可，推荐使用qBittorrent，下载速度会比直链快，毕竟是开源软件（去镜像站下载也可以）

# 安装
保持安装u盘不用拔出，重启进入bios设置u盘启动优先  
再次重启就是arch安装u盘的界面了

Q：但是我的archlinux安装盘在不断重启啊  
A：小问题，在grub选择页面按 `e` 然后在 `linux` 那一行的最后加上`nomodeset`来设置内核不加载容易出错的内核模块  

一切顺利的话就会看到红色的`archiso`
接着我们输入：
```
archinstall  
```
注意：archinstall 仅支持 UEFI 系统

按照安装脚本给出的提示即可，这里提一些值得注意的  
* ssd固态硬盘用户建议使用brtfs文件系统
* 时区手动输入的话是`Asia/Shanghai`
* 要让你的系统支持休眠/睡眠的话就要让zram的大小`>=`物理内存  

安装完成后他会询问你是否要立即重启，此时你安装的如果是nvidia的闭源驱动，我们不要立刻重启  
在`/etc/modprobe.d/`下面新建`.conf`结尾的配置文件屏蔽内核中自带的`nouveau`开源驱动  
操作如下：  
```
cd /etc/modprobe.d/  
nano blacklist.conf #系统只识别.conf的文件，所以文件名随意  
```

添加内容如下：  
```
 blacklist nouveau  
 blacklist lbm-nouveau  
 options nouveau modeset=0  
 ```

即可重启
# 配置
## amdgpu核芯显卡+nvidia私有驱动（非独显直连）配置混合模式
并不需要bumblebee/optimus-manager/nvidia-xrun这种切换工具  
根据[PRIME-Archwiki英文页面](https://wiki.archlinux.org/title/PRIME#Configuration)的说法  

先识别集成GPU BusID 得到以下结果  
```
lspci | grep -i VGA
-----------------------------------------
01:00.0 VGA compatible controller: NVIDIA Corporation GA106M [GeForce RTX 3060 Mobile / Max-Q] (rev a1)
06:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Cezanne (rev c5)
```
然后编辑/etc/X11/xorg.conf  
```
/etc/X11/xorg.conf
-----------------------------------------
Section "ServerLayout"
        Identifier "layout"
        Screen 0 "amdgpu"
        Inactive "nvidia"
        Option "AllowNVIDIAGPUScreens"
EndSection

Section "Device"
        Identifier "nvidia"
        Driver "nvidia"
EndSection

Section "Screen"
        Identifier "nvidia"
        Device "nvidia"
EndSection

Section "Device"
        Identifier "amdgpu"
        Driver "modesetting"
        BusID "PCI:6:0:0"
EndSection

Section "Screen"
        Identifier "amdgpu"
        Device "amdgpu"
EndSection

```


## lightdm黑屏 在其他显示器显示
等我装完重启进入桌面后通过运行`nvidia-setting`报错发现我的RTX 3060并没有工作  
bing搜索得解决办法 于是使用`nvidia-xconfig`配置`/etc/X11/xorg.conf`  
遂重启，然后这个archlinux怎么又双叒黑屏寄了  
我一开始还以为是amdgpu+nvidia的显卡配置问题  
后来才发现是lightdm这玩意压根就不支持多显示屏（我使用xfce4桌面，默认使用lightdm作为登录管理器），这就导致了我的登录界面在一颗虚空显示屏上面

解决方法：[如何强制多个显示器为LightDM设置正确的分辨率？](https://qastack.cn/ubuntu/119843/how-to-force-multiple-monitors-correct-resolutions-for-lightdm)  
我的方法：垃圾lightdm狗都不用 遂卸载lightdm 每天使用`startx`启动桌面
## apu加nvidia用户在双屏的时候只有单屏显示  

这个东西我看了半天xorg的英文文档和archlinux的[Amdgpu](https://wiki.archlinux.org/title/AMDGPU_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#Xorg_configuration)页面 搞得我快一口老血吐出来  

按照archwiki的说法要创建`/etc/X11/xorg.conf.d/20-amdgpu.conf`添加如下内容：  
```
 Section "Device"  
     Identifier "AMD"  
     Driver "amdgpu"  
 EndSection
```
我也给arcwiki在nvidia页面贡献了此次issue
## 安装输入法
我选择fcitx 毕竟这个项目的名字意思都是中文输入法  
安装fcitx5：  
```
sudo pacman -Sy fcitx5 fcitx5-chinese-addons fcitx5-configtool fcitx5-gtk fcitx5-rime fcitx5-qt
```

配置过程参见[archwiki_fcitx5](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85)  
## 安装yay以及timeshift备份管理软件
安装yay  
```
sudo pacman -Sy archlinuxcn-mirrorlist
sudo pacman -Sy yay
```
yay安装时如果报错就尝试  
原理见[GnuPG-2.1 与 pacman 密钥环](https://www.archlinuxcn.org/gnupg-2-1-and-the-pacman-keyring/)
```
pacman -Syu haveged
systemctl start haveged
systemctl enable haveged

rm -fr /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman-key --populate archlinuxcn
```

安装timeshift备份工具  
timeshift比windows那羸弱的系统还原点高到不知道哪里去了  
rsync对比更改文件建立镜像就可以一键回滚，再也不用担心系统滚挂重装的烦恼了
```
yay -S timeshift
```
## 更换内核
linux-zen内核是g胖支持的，对steam上的wine游戏经过了[特殊的优化](https://github.com/ValveSoftware/Proton/issues/3706#issuecomment-636632984)  
```
sudo pacman -S linux-zen linux-zen-headers
```
`内核名-headers`这种包一定要装哦 不然定制内核的模块编译不进去的  

> DKMS，即 Dynamic Kernel Module System。可以使内核变更（如升级）后自动编译模块，适配新内核。

要注意使用非默认内核时部分包是不支持的需要安装dkms包  
比如:  
`nvidia`是针对默认内核的，定制内核需要安装`nvidia-dkms`  
`virtualbox-host`是针对默认内核的，定制内核需要安装`virtualbox-host-dkms`  

如果你直接换内核而不换nvidia驱动的话你在启动图形界面的时候就会发现：  
```
xf86EnableIOPorts: failed to set IOPL for I/O (Operation not permitted)
```
（就像我一样

## 挂起后无法唤醒
~~[在这里千辛万苦找到的解决办法](https://bbs.archlinux.org/viewtopic.php?id=271468)~~  
~~创建`/etc/modprobe.d/`添加如下内容即可:~~  
~~`blacklist nvidia-uvm`~~
  
2022.02.04更新  
经过论坛询问，解决挂起后无法唤醒的要看nvidia的文档，详见[这里](https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html)  
```
    sudo systemctl enable nvidia-suspend.service

    sudo systemctl enable nvidia-hibernate.service

    sudo systemctl enable nvidia-resume.service

```

2022.02.06更新
我是amd笔记本，这样的休眠黑屏的bug大概率是5.15之前的linux内核bug 现在已经修复，不用管了

# dwm使用配置
我懒得折腾直接使用默认官方配置
```
窗口管理器：dwm  
终端：st，tilda  
应用启动器：dmenu  
系统托盘：trayer  
    托盘应用：nm-applet（网络管理器）
            pasystray（声音管理器）  
            blueman-applet（蓝牙）  
            fcitx5（输入法）
            cfw（魔法上网）  
```

# KVM虚拟机折腾
  
```
# 检测宿主机cpu是否支持虚拟化，如果flags里有vmx 或者svm就说明支持VT
grep -E "(vmx|svm)" --color=always /proc/cpuinfo

# 检查内核的KVM和VirtIO模块是否可用
zgrep KVM /proc/config.gz
zgrep VIRTIO /proc/config.gz 

# 查看内核模块是否装载
lsmod | grep kvm 
lsmod | grep virtio

# 手动加载内核模块
sudo modprobe virtio

# 当前用户加入组kvm
sudo usermod -a -G kvm YOURSUSERNAME

# 安装qemu以及图形化客户端
sudo pacman -S qemu libvirt ovmf virt-manager

# 为kvm开启网络支持
sudo pacman -S ebtables dnsmasq bridge-utils openbsd-netcat

# 启动服务
sudo systemctl enable libvirtd
sudo systemctl start libvirtd

sudo systemctl enable virtlogd
sudo systemctl start virtlogd
```
* 注意事项  
每次启动虚拟机的时候都要手动启动虚拟机的网络，很麻烦  
我们在virt-manager里面右键`QEMU/KVM`，点击`虚拟网络`在`自动启动`这一行勾选`引导时`  

## 安装windows11的时候要做的工作  
在`overview`里面选择带有`secboot`的固件来支持windows11需要的安全启动  
接着安装`swtpm`软件包（详见各发行版的包名）然后增加TPM设备，选择`Emulated`，`TIS`和`2.0`

## 为kvm里的win10安装驱动
首先：
```
sudo pacman -S virtio-win
```
然后安装好win10后进入计算机找到已经挂载的磁盘，运行最下面的驱动安装程序即可，安装好就能体验到自动调整窗口大小和kvm的丝滑了

## 接下来为kvm添加文件拖拽共享功能  
csdn上的各种不靠谱还互相抄，我在[ubuntu论坛](https://askubuntu.com/questions/885264/kvm-copy-drag-and-drop-files-between-ubuntu-host-to-windows-7-guest)找到了  
要下载[spice驱动](https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe)随后重启虚拟机即可完美实现拖放和剪贴板共享

## gnome NVIDIA使用wayland
`sudo nano /etc/default/grub`  
```
GRUB_CMDLINE_LINUX_DEFAULT="nvidia-drm.modeset=1"
```
  

`sudo nano /etc/mkinitcpio.conf`
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

运行  
`sudo mkinitcpio -P`  
`sudo update-grub`或`sudo grub-mkconfig -o /boot/grub/grub.cfg`
  
其次要看 `/etc/gdm/custom.conf` 里面 `WaylandEnable=false` 是不是被注释掉的（默认应该是）  
  
最可能的是 udev 规则，GDM 带了一个 /usr/lib/udev/rules.d/61-gdm.rules 文件，里面有这么几行：
```
# disable Wayland on Hi1710 chipsets
ATTR{vendor}=="0x19e5", ATTR{device}=="0x1711", RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
# disable Wayland when using the proprietary nvidia driver
DRIVER=="nvidia", RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
# disable Wayland if modesetting is disabled
IMPORT{cmdline}="nomodeset", RUN+="/usr/lib/gdm-runtime-config set daemon WaylandEnable false"
```
第二行表示如果检测到 NVIDIA 闭源驱动就关闭 Wayland，第三行则是如果没开启 Kernel Mode Setting 也关掉 Wayland。

我猜大部分用户都没有第一行的设备，所以我建议直接 ln -s /dev/null /etc/udev/rules.d/61-gdm.rules 屏蔽掉这个文件。




> ### 参考资料
> [arch简明指南](https://arch.icekylin.online)  
[archlinux wiki](https://wiki.archlinux.org/)  
[NVIDIA (简体中文) ](https://wiki.archlinux.org/title/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AF%B9%E4%BA%8E%E5%90%8C%E6%97%B6%E6%8B%A5%E6%9C%89amd%E6%A0%B8%E6%98%BE%E5%92%8Cnvidia%E7%8B%AC%E7%AB%8B%E6%98%BE%E5%8D%A1%E7%9A%84%E7%94%A8%E6%88%B7)  
[GRUB (简体中文) - ArchWiki](https://wiki.archlinux.org/title/GRUB_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))  
[mkinitcpio (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Mkinitcpio_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%A8%A1%E5%9D%97%EF%BC%88MODULES%EF%BC%89)  
[Kernel (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Kernel_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))  
[xinit (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Xinit_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E9%85%8D%E7%BD%AE)  [Xorg (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Xorg_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E9%85%8D%E7%BD%AE)
