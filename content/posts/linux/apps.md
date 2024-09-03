---
title: "常用开源软件"
date: 2023-02-22T11:11:27+08:00
draft: false
tags: ["linux"]
ShowToc: true
---

> 我的电脑双硬盘双系统，使用Windows 11 + ArchLinux  
> 在Windows上使用 scoop + winget / Archlinux 下使用 paru+pacman  
> 我尽可能使用开源/免费

# 全平台

- 网络加速：[Watt Toolkit](https://steampp.net/)  
  提供了多网站的网络加速 不局限于steam 我一般用来加速github 和 steam
- 编辑器：[Neovim](https://neovim.io/)  
  编辑器之神 没什么好说的
- 本地视频播放： [mpv](https://mpv.io/)  
  轻巧的一款本地播放器 支持字幕 我拿它来放电影看
- 视频录制：[OBS](https://obsproject.com/)
- 手机投屏：[scrcpy](https://github.com/Genymobile/scrcpy)  
  基于命令行 连上手机开启adb调试以后敲命令就行 没有多余的依赖
- 流媒体：[YesPlayMusic（网易云）](https://github.com/qier222/YesPlayMusic),[Cider(Apple Music)](https://cider.sh/),Spotify  
  用 YesPlayMusic 多一点，因为它有原版网易云不支持的“连接到Last.FM”功能
- GIT：[LazyGit](https://github.com/jesseduffield/lazygit)  
  我的水平还不足以完全开发这个软件 不常用
- 配置文件管理：[GNU STOW](https://www.gnu.org/software/stow/)  
  非常好用的配置文件管理，用软连接的方式链接到文件所在位置，源文件集中在一起，非常方便
- 系统监视器：[Btop/Bottom](https://github.com/aristocratos/btop)
- 截图：[Flameshot](https://flameshot.org/)  
  上游还没修Wayland下面的bug TAT
- 种子下载：[Motrix](https://motrix.app/)  
  非常好用 自带tracker list 即使它是electron我用能接受
- 代码分享：[Silicon](https://github.com/Aloxaf/silicon)
- 终端：[wezterm](https://wezfurlong.org/)
- 密码管理：[KeepassXC](https://keepassxc.org/)
- 隔空投送：[Localsend](https://github.com/localsend/localsend)  
  好用，尤其是在AP隔离的校园网环境下

# Linux

Linux only 的软件我放在了我的[Dotfile 仓库的介绍里面](https://github.com/hydroakri/dotfiles)

# Windows

- 把Windows 11 改为10的模样：[ExplorerPatcher](https://github.com/valinet/ExplorerPatcher)
- 使游戏窗口全屏显示：[Magpie](https://github.com/Blinue/Magpie)
- 管理“此电脑”里删不掉的流氓“快捷方式”：[MyComputerManager](https://github.com/1357310795/MyComputerManager)
- 平铺管理器：[komorebi](https://github.com/LGUG2Z/komorebi)
- 系统优化：[optimizer](https://github.com/hellzerg/optimizer), [Dism++](https://github.com/Chuyu-Team/Dism-Multi-language)
- 让Windows拥有macOS的触发角功能: [hotcorner](https://github.com/misterchaos/hotcorner)
- Windows免费激活：[Microsoft-Activation-Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts)
- 系统信息：[systeminformer](https://github.com/winsiderss/systeminformer)
- 桌面文件夹整理：[TaskbarX](https://github.com/ChrisAnd1998/TaskbarX)
- 内存优化：[memreduct](https://github.com/henrypp/memreduct)
- WSL管理：[LxRunOffline](https://github.com/DDoSolitary/LxRunOffline)
- BTRFS: [btrfs](https://github.com/maharmstone/btrfs)
