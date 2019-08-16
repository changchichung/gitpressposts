---
title: "用DELL 6 i/R 建立RAID，並在上面安裝ubuntu 18.04 "
date: 2019-01-16T16:17:05+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['ubuntu']
author: "Eric Chang"
---

買了一張 DELL 6/iR 低階的raid 卡

來測試把系統裝在硬體做的RAID上，結果沒想到居然不能開機...
<!--more-->

都2019年了， DELL 6/iR 這張卡出了快十年了，不懂為什麼在安裝過程都正常

但是裝完之後，都會發生 " no boot device "的錯誤

測試過 ubuntu 16.04 / 18.04 , Debian 9 都是一樣的問題

翻了很久google，發現有人提到要修改一個grub的設定

先裝完系統，然後改用 ubuntu 18.04 Live DVD 開機

然後開啟 terminal

依序執行

```
sudo mount /dev/sdf1 /mnt (RAID磁碟代號不一定是 /dev/sdf 先用fdisk -l 確認)

sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt

```

chroot 完成之後，再進行下面的修改
```
vi /etc/default/grub
找到
GRUB_CMDLINE_LINUX=""
修改成
GRUB_CMDLINE_LINUX="rootdelay=90"
```

存檔後離開，再更新 grub

```
update-grub
```

然後重新開機，開機時間會比較久一點，但是這樣就可以正常開機了。

還是不懂，這到底是 ubuntu的問題，還是raid Controller的問題？還是 grub 的問題？
