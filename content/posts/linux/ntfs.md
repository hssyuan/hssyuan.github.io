---
title: "NTFS 的一些小技巧"
date: 2024-09-02T20:48:27+08:00
draft: false
tags: ["linux"]
---

鉴于内核在 5.15 之后合并了NTFS3的内核驱动使得读写能力大幅提升，可以把ntfs-3g扔掉了

```rules
# sudo vi /etc/udev/rules.d/ntfs3_by_default.rules

SUBSYSTEM=="block", ENV{ID_FS_TYPE}=="ntfs", ENV{ID_FS_TYPE}="ntfs3" ENV{UDISKS_MOUNT_OPTIONS_DEFAULTS}="uid=1000,gid=1000,rw,user,exec,umask=000"
```

注：选项是把NTFS3作为默认挂载驱动，并且挂载选项使用了[proton建议的设置](https://github.com/ValveSoftware/Proton/wiki/Using-a-NTFS-disk-with-Linux-and-Windows)
