---
title: "Install Asus 10G NIC XG-C100C in Proxmox"
date: 2019-06-17T13:20:57+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: [Proxmox]
tags: [proxmox]
author: "Eric Chang"
---

前幾天接的一個case

因為費用的關係，所以沒有考慮用傳統定義上的伺服器(DELL R640)

改採用比較高階一點的洋垃圾，規格大概是 Intel E5-2680V2 x2 + 64G RAM + 128G SSD x2 (OS) + 960G SSD x4 (raid 10 , zfs)

storage 選擇QNAP NAS  TS-932X + 960G SSD x 4 (raid 10 , NFS) + QNAP 10G Switch QSW-1280C-8C

既然storage這邊選用了10G的機種，伺服器上當然也要增加10G網卡

一樣，成本考量，就不用INTEL 了，買了這張 ASUS 10G 網卡

<!--more-->
https://24h.pchome.com.tw/prod/DRAF01-A90088CKR

結果.... debian9 預設抓不到這張卡啊！

本來想說要退貨了，想想還是先google 好了，還真的有人碰到一樣的問題

底下就大概說一下怎麼解決

###安裝相關套件###

登入proxmox 主機後，執行以下指令安裝套件
```
apt install linux-headers build-essential git make gcc
```

然後馬上就卡關了 XD

```
root@pve:~# apt install linux-headers
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Package linux-headers is a virtual package provided by:
  pve-headers-4.15.3-1-pve 4.15.3-1
  pve-headers-4.15.18-9-pve 4.15.18-30
  pve-headers-4.15.18-8-pve 4.15.18-28
  pve-headers-4.15.18-7-pve 4.15.18-27
  pve-headers-4.15.18-6-pve 4.15.18-25
  pve-headers-4.15.18-5-pve 4.15.18-24
  pve-headers-4.15.18-4-pve 4.15.18-23
  pve-headers-4.15.18-3-pve 4.15.18-22
  pve-headers-4.15.18-2-pve 4.15.18-21
  pve-headers-4.15.18-15-pve 4.15.18-40
  pve-headers-4.15.18-14-pve 4.15.18-39
  pve-headers-4.15.18-13-pve 4.15.18-37
  pve-headers-4.15.18-12-pve 4.15.18-36
  pve-headers-4.15.18-11-pve 4.15.18-34
  pve-headers-4.15.18-10-pve 4.15.18-32
  pve-headers-4.15.18-1-pve 4.15.18-19
  pve-headers-4.15.17-3-pve 4.15.17-14
  pve-headers-4.15.17-2-pve 4.15.17-10
  pve-headers-4.15.17-1-pve 4.15.17-9
  pve-headers-4.15.15-1-pve 4.15.15-6
  pve-headers-4.15.10-1-pve 4.15.10-4
  pve-headers-4.13.8-3-pve 4.13.8-30
  pve-headers-4.13.8-2-pve 4.13.8-28
  pve-headers-4.13.8-1-pve 4.13.8-27
  pve-headers-4.13.4-1-pve 4.13.4-26
  pve-headers-4.13.3-1-pve 4.13.3-2
  pve-headers-4.13.16-4-pve 4.13.16-51
  pve-headers-4.13.16-3-pve 4.13.16-50
  pve-headers-4.13.16-2-pve 4.13.16-48
  pve-headers-4.13.16-1-pve 4.13.16-46
  pve-headers-4.13.13-6-pve 4.13.13-42
  pve-headers-4.13.13-5-pve 4.13.13-38
  pve-headers-4.13.13-4-pve 4.13.13-35
  pve-headers-4.13.13-3-pve 4.13.13-34
  pve-headers-4.13.13-2-pve 4.13.13-33
  pve-headers-4.13.13-1-pve 4.13.13-31
  pve-headers-4.10.8-1-pve 4.10.8-7
  pve-headers-4.10.5-1-pve 4.10.5-5
  pve-headers-4.10.17-5-pve 4.10.17-25
  pve-headers-4.10.17-4-pve 4.10.17-24
  pve-headers-4.10.17-3-pve 4.10.17-23
  pve-headers-4.10.17-2-pve 4.10.17-20
  pve-headers-4.10.17-1-pve 4.10.17-18
  pve-headers-4.10.15-1-pve 4.10.15-15
  pve-headers-4.10.11-1-pve 4.10.11-9
  pve-headers-4.10.1-2-pve 4.10.1-2
You should explicitly select one to install.

E: Package 'linux-headers' has no installation candidate
root@pve:~# 
```

### 改用以下套件名稱安裝###
```
apt install pve-headers-4.15.18-15-pve build-essential git make gcc
```

第一個套件會隨著pve版本不同有所變化，所以要看當下執行時，系統提供的訊息來決定

我是用清單內最新的版本

接著把驅動程式 clone 回來

```
git clone https://github.com/Aquantia/AQtion.git
```

參考 README

https://github.com/Aquantia/AQtion/blob/master/README.txt

進入目錄後開始編譯
```
make
```

編譯完，應該就能看到網卡了，不過我是還有重開機

順便貼一下10G網路的速度，感覺還真的.....不怎麼樣 ....

![](https://i.imgur.com/yM8HsSi.png)

不過在做 backup/restore 的時候，感覺是比之前其他沒有10G環境要快

反正這樣子買下來的硬體設備也不算太貴(比一台新的R640還便宜)

就先這樣子跑吧，至於洋垃圾的穩定度，就觀察看看吧..


