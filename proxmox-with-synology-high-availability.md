---
title: "[筆記] 測試 proxmox 存取由 synology HA cluster 分享的NFS 目錄 / Proxmox With Synology High Availability"
date: 2020-01-17T12:20:33+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-18.jpg
tags: ['synology','proxmox','high Availability']
author: "Eric Chang"
keywords:
  - synology
  - proxmox
  - 'high availability'
---

前幾天修復了因為intel cpu bug 導致無法使用的 synology DS415+

詳情請看 https://h.cowbay.org/post/first-try-synology-ha/

今天趁尾牙前夕，手邊沒啥要緊事

就來玩玩看promox 加上 synology high availability 再加上 NFS share 的環境

<!--more-->

先上架構圖

![](https://i.imgur.com/k7IDZ4Y.png)

架構很簡單，NAS設定一組NFS share， proxmox mount 進來，然後開一台VM在NFS 上

主要來談談proxmox 在碰到synology high availability 切換狀態、遇上腦裂(brain split)時候的狀況

觸發 brain split (說真的，我覺得腦裂很難聽 ...)的情況，在上面連結那篇文章裡面有提到，就不多說了

來講講後續的狀況

發生 brain split 時，可以預期管理者會登入管理界面去修復

關於修復brain split 可以看看群暉的這篇文章

https://www.synology.com/zh-tw/knowledgebase/DSM/help/HighAvailability/split_brain

而我選擇的是 [將兩台伺服器一同保留於叢集中] 

在進行修復的過程中，會發現VM這邊會變成 read only 

聽起來很合理，畢竟在修復時，所有服務幾乎都是停擺

但是呢，等到修復完成後，VM還是read only ，這就有點奇怪了

有跟群暉客服反應過這個狀況

所以在修復完成之後，在proxmox server 這邊直接對NFS 存取做測試

去下載一個template 是 OK 的，在console 裡面直接在NFS touch file 也是可以的

所以Synology high availability 是有正常發揮作用

而promox 這邊，在synology恢復之後，也可以正常存取NFS ，所以也沒有問題

那問題就是在VM裡面了，當發生了某些狀況，讓系統進入read only ，就必須透過reboot 才能解決

或者是看看這個指令用fsck去檢查filesystem 看看有沒有幫助

```
sudo fsck -Af -M
```


