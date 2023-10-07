---
title: "linux 使用 doh"
date: 2023-08-09T17:26:27+08:00
draft: false
tags: ["linux"]
---

在比较了[archwiki](https://wiki.archlinux.org/title/Domain_name_resolution)关于dns的介绍后  
在smartdns/coredns/unbound/dnsencrypt-proxy之中选择了后者  
coredns和unbound对于桌面linux用户而言过于复杂且不好配置  
而smartdns面对老前辈dnsencrypt-proxy而言缺少了自动获取doh源的功能

# 安装
1. 首先`sudo -s`进入root shell  
然后`ss -lp 'sport = :domain'`查看端口是否被占用  
确保`127.0.0.1:domain`没有被占用即可  
如果有占用需要停止目前使用的dns服务  
例如`systemctl disable systemd-resolved`


2. 然后根据发行版安装二进制包  
配置文件的位置在`/etc/dnscrypt-proxy/dnscrypt-proxy.toml`
启动服务`systemctl enable dnscrypt-proxy.service --now`

3. 让dns服务器指向本地的dnscrypt-proxy
修改`/etc/resolv.conf`
```/etc/resolv.conf
nameserver ::1
nameserver 127.0.0.1
options edns0
```
然后`chattr +i /etc/resolv.conf`来放置resolv.conf被networkmanager或其他网络服务覆写  

打开浏览器并前往[dns leak test](https://www.dnsleaktest.com/)并进行扩展测试,如果结果显示您在配置文件中设置的服务器,则表示dnscrypt-proxy正在工作,否则出现问题。

