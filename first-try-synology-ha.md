---
title: "[筆記] 第一次玩 Synology High Availability / first try synology high availability"
date: 2020-01-10T09:48:18+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-13.jpg
tags: ['synology']
author: "Eric Chang"
keywords:
  - synology
  - high availability
---

上禮拜，原本擔任 proxmox cluster 的主要 storage 的 ds415+ 掛點了

原因應該就是之前的 intel c2000 series cpu 的 bug

只是不知道為什麼這台兩三年來都沒有關機的NAS

比其他三台多撐了那麼久 (已經有兩台送修回來，一台也是同樣症狀，被放在一邊)

趁著這次機會，看看網路上說的換電阻大法有沒有用！

如果有用，就拿這兩台來玩玩 synology high availability !

<!--more-->

先要感謝這一篇的作者！

https://www.mobile01.com/topicdetail.php?f=494&t=5600042

在網路上訂了一大包的 1/4 w 100Ω 的電阻 (100個才70塊，運費都要60了)

照著上面那篇的作法，把電阻焊上去，NAS就順利開機了！

__

架構圖很簡單，只是在做測試而已，又是第一次玩，先不要搞得太複雜

![](https://i.imgur.com/k7IDZ4Y.png)

流程大致如下

設定好NAS Cluster 之後，建立NFS 服務

然後在proxmox 主機上掛載這個NFS 空間

接著在proxmox 上建立一台 VM ，存放在NFS 空間上

在這台VM裡面持續 ping NAS cluster VIP 192.168.11.85

接著拔掉 192.168.11.87 的兩條網路線，模擬NAS cluster 的主伺服器掛點的狀況

這時候VM 還活著，可以正常建立、刪除、檢視檔案，然後 ping 192.168.11.85 也還持續著

NAS的告警信件也正常發出

08:53 NAS High Availability 叢集 ds415cluster 已執行自動故障轉移。 [詳細資訊：無法偵測到 hqs087 (主伺服器)]
08:58 NAS High Availability 叢集 ds415cluster 狀態異常 [詳細資訊：無法偵測到 hqs087 (副伺服器)]

9:08 接回hqs087的網路線

9:09 收到信件 NAS High Availability 叢集 ds415cluster 停止正常運作 [詳細資訊：Split-brain 錯誤]

登入管理界面(192.168.11.85:5000) ，操作 HA ，選擇恢復

這時候開始，VM 的檔案系統變成是 read only

雖然還活著，但是已經無法建立、刪除檔案，連 cat /var/log/syslog 也會卡住

9:14 VIP NAS cluster 恢復連線，本來卡住的 cat /var/log/syslog 也可以正常顯示內容了

但是系統還是 read only，reboot VM 之後才恢復正常。

有幾個問題

* split brain 錯誤

這個問題我想應該是因為只有兩台組成clsuter 造成的

如果有第三臺加入，應該就不會有這個split brain 的問題

* VM變成 read only

這個我就不知道為什麼了，照理說NAS Cluster 已經開始在恢復

在我的觀念裡，應該要能夠「正常」的持續服務

但是VM變成 read only ，而且必須要重新開機才能解決

那這樣NAS Cluster 等於沒有太大作用呀..

來問問看群暉客服好了















