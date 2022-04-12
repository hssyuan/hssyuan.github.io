---
title: "基于KVM的虚拟显卡透传技术"
date: 2022-04-12T22:56:27+08:00
draft: false
tags: ["linux"]
---

# 0.准备工作&检查
显卡直通的前提条件是：  
* NVIDIA 独立显卡本身要具有视频输出功能  
* 机身上至少有一个连接到独立显卡的视频接口  
* 安装kvm虚拟机，具体步骤参照本博客archlinux安装或archlinux wiki

强烈建议阅读本文时参照下面几篇文章：  
[PCI_passthrough_via_OVMF_ARCHLINUX wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E7%A1%AC%E4%BB%B6%E8%A6%81%E6%B1%82)  
[Optimus MUXed 笔记本上的 NVIDIA 虚拟机显卡直通](https://lantian.pub/article/modify-computer/laptop-muxed-nvidia-passthrough.lantian/)  
[vanities/GPU-Passthrough-Arch-Linux-to-Windows10
Public](https://github.com/vanities/GPU-Passthrough-Arch-Linux-to-Windows10)  
[
KVM-GPU-Passthrough](https://github.com/BigAnteater/KVM-GPU-Passthrough)  
[Optimus笔记本虚拟机显卡直通](https://www.bwsl.wang/script/119.html)

# 1.1开始  
## 1.1隔离独显  
### 1.1.1引导镜像的编辑
运行 ` lspci -nn | grep NVIDIA `，获得类似如下输出：
```
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA106M [GeForce RTX 3060 Mobile / Max-Q] [10de:2520] (rev a1)
01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:228e] (rev a1)
```
编辑或创建`/etc/modprobe.d/vfio.conf`，内容书写 
```
options vfio-pci ids=10de:2520,10de:228e
```
修改`/etc/mkinitcpio.conf`，在**MODULES**里增加
```
vfio_pci vfio vfio_iommu_type1 vfio_virqfd
```  
还需要确保`/etc/mkinitcpio.conf`之中有`modconf`  
之后执行`sudo mkinitcpio -P`更新initramfs  
### 1.1.2引导时内核参数的编辑
为了使iommu加载，还需要在引导选项中  
修改`/etc/default/grub`  
对于intel用户：
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet ... intel_iommu=on"
```
对于amd用户  
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet ... amd_iommu=on"
```
然后构建grub引导
`sudo grub-mkconfig -o /boot/grub/grub.cfg`

### 1.1.3重启后检查
运行`lspci -nnk`若输出中`Kernel driver in use`一行为`vfio-pci`则万事具备  
```
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GA106M [GeForce RTX 3060 Mobile / Max-Q] [10de:2520] (rev a1)
	DeviceName: NVIDIA Graphics Device
	Subsystem: Hewlett-Packard Company Device [103c:88d1]
	Kernel driver in use: vfio-pci
	Kernel modules: nouveau, nvidia_drm, nvidia
01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:228e] (rev a1)
	Subsystem: Hewlett-Packard Company Device [103c:88d1]
	Kernel driver in use: vfio-pci
	Kernel modules: snd_hda_intel
```

## 1.2准备一个虚拟机
之前文章已经说过了，在这不赘述了，在这里写一些可能遇到的问题  
### 1.2.1 BIOS的选择
在bios上建议选择UEFI,对显卡的支持好一些  
在**安装前**选择安装前配置，选择`概览`，`芯片组`选择`Q35`，`固件`选择`/usr/share/edk2-ovmf/x64/OVMF_CODE.fd`  
接着进入安装windows,别忘了给windows打上`virtio-win`的驱动
### 1.2.2 如果你已经给windows安装了上了传统legacy bios
进入windows,以管理员权限运行  
```
mbr2gpt /convert /allowfullOS
```
之后将kvm的`概览`的xml中的`os`字段改为以下模样  
```
<os>
    <type arch="x86_64" machine="pc-q35-6.2">hvm</type>
    <loader readonly="yes" type="pflash">/usr/share/edk2-ovmf/x64/OVMF_CODE.fd</loader>
    <nvram>/var/lib/libvirt/qemu/nvram/win10_VARS.fd</nvram>
    <boot dev="hd"/>
</os>
```
## 1.3 虚拟机上的操作
先不着急添加直通显卡，先进入虚拟机安装对应的显卡驱动，然后重启  
在虚拟机上添加`PCI主机设备`把刚刚做过手脚的gpu选择上  
不出意外的话重启你竟然发现原来丝滑的鼠标突然不能用了  
已知的解决方法：  
* 某宝购买kvm切换器，然后在 Virt-Manager 里选择添加硬件（Add Hardware） - USB 宿主设备（USB Host Device），选择你的鼠标键盘即可  
* Looking-glass或者scream,请自行参照官方文档安装配置  
* 笔记本电脑有一套外接键鼠的话直接直通这一套键鼠，linux上使用触摸板和自带键盘  

# 虚拟机性能优化  
## cpu 优化
在“CPUs”部分，将您的 CPU 型号更改为`host-passthrough`
然后打开`手动设置CPU拓扑`，保持`套接字`为`1`（这是为了让cpu能够减少内存交换的次数）  
## io 优化  
进入“添加硬件”并为“VirtIO SCSI”型号的**SCSI驱动器添加控制器**  
然后然后将默认 SATA 磁盘更改为SCSI磁盘，该磁盘将绑定到所述控制器。
## grub里面添加hugepage参数
## cpu固定