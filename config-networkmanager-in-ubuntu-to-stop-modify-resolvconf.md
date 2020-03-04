---
title: "[筆記] 設定ubuntu 18.04 的NetworkManager config 不要更改 /etc/resolv.conf"
date: 2020-03-04T16:38:55+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-17.jpg
tags: ['筆記','Networkmanager','resolv.conf']
author: "Eric Chang"
keywords:
  - resolv.conf
  - networkmanager
---

ubuntu 18.04 的 DNS 設定很煩

系統預設會用NetworkManager 去管理

然後NetworkManager 又很「靈活」的許多種修改 /etc/resolv.conf 的方式

之前都是很粗暴的停用 NetworkManager

但是用筆電的user 又需要用 NetworkManager 來管理無線網路

今天找了一下文件，讓NetworkManager 可以執行，卻不會去異動 /etc/resolv.conf

<!--more-->

主要參考這篇文件

https://developer.gnome.org/NetworkManager/stable/NetworkManager.conf.html

看一下 dns/rc-manager 這兩個部份

然後修改 /etc/NetworkManager/NetworkManager.conf

```
[main]
plugins=ifupdown,keyfile
dns=none
rc-manager=unmanaged

[ifupdown]
managed=false

[device]
wifi.scan-rand-mac-address=no
```

主要就加入第三行和第四行

接著安裝 resolvconf 這個套件
```
sudo apt install resolvconf
```

修改resolvconf 的config

```
sudo vim /etc/resolvconf/resolv.conf.d/head
加入以下內容

nameserver 168.95.1.1
nameserver 8.8.8.8
```

然後重新啟動 NetworkManager 還有 resolvconf 或者重新開機

就可以用 resolvconf 來管理 /etc/resolv.conf 

不會再發生DNS 被改成 127.0.0.53 這種怪東西了


