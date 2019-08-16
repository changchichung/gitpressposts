---
title: "[筆記] Freenas Smartd 啟動失敗 Smartd Failed to Start in Freenas"
date: 2018-12-13T17:40:20+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-2.jpg"
categories: ['筆記']
tags: ['freenas']
author: "Eric Chang"
---

這兩天在弄兩台Freenas ，準備當作Proxmox 的Storage & Server Backup

因為伺服器的限制，只能接六個SATA，我接了六個2T的硬碟做raid10

然後把Freenas 安裝在隨身碟上

不過會一直出現Smartd failed to start 的錯誤訊息

<!--more-->

![Freenas smartd failed to start](https://i.imgur.com/mj6lCc7.png)

翻了一下論壇，發現因為系統安裝在隨身碟上

然後預設會開啟隨身碟的smart 

可是這樣反而會造成錯誤

所以只要去 Storage --> View Disks 找到隨身碟的代號

然後點兩下進去編輯，把 enable S.M.A.R.T 關閉

![disable SMART on USB FLASH](https://i.imgur.com/WnvsQie.png)

接著再去啟動 SMART ，就不會有錯誤訊息了
