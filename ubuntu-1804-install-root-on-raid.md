---
title: "Ubuntu 1804 Install Root on Raid"
date: 2019-01-16T09:58:50+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['ubuntu','raid']
author: "Eric Chang"
---

最近在弄一台機器，想要把ubuntu 18.04 安裝在software raid上

因為新開的機器大部分都是在proxmox上，所以很少碰實體機器了

結果在安裝過程中，做raid碰到一些問題，來紀錄一下

<!--more-->

要先說明 Ubuntu 提供的ISO類型，這會牽涉到後續裝raid

底下這是ubuntu 網頁上的ISO列表

![ubuntu iso list](https://i.imgur.com/32JIHL0.png)

大致上分為 Desktop/live-server 兩種

因為我要裝的是server，所以我一開始當然是選live-server

但是用這個ISO開機，要設定software raid時，會出現警告訊息

提示不可以把所有的分割區都指定給 RAID/LVM ，這樣會沒有地方可以放 /boot

錯誤如圖

![create root on raid error](https://i.imgur.com/uhSpn6w.png)


所以我很「鄉愿」的，那就切一個/boot 給它用，算是暫時解決這問題 XD

但是這樣的作法，總有一天會出事

因為如果這個 /boot 掛了，雖然底下的系統有做mirror

但還是不能開機，那這樣做raid根本沒有意義啊！

所以研究了兩天，發現一個很重要的事情

我根本就抓錯ISO了啊！！！！！！！

會這樣想是因為中間有其他task在裝debian9

一開始也是抓live-dvd版本

但是這個版本沒有辦法自訂要安裝哪些套件，所以預設安裝完會包含windows manager、office、字型等等

加起來總共5.x G ....

然後我還要手動移除這些套件，這不是脫褲子放屁嗎？

翻了一下google，發現是因為ISO的關係，要去下載netinst的ISO

才能在安裝過程中自訂套件

從這邊延伸到ubuntu的問題

會不會是我也抓錯ISO了呢？

再次google相關訊息，果然ubuntu也有類似的netboot ISO

![ubuntu mini iso](https://i.imgur.com/G2ImxhQ.png)

檔案很小，只有60M左右，趕快下載來安裝！

這次果然可以在安裝過程中，順利設定software raid，並且掛載在 / 根目錄底下進行安裝

## BUT .... 對，永遠少不了這個BUT

安裝過程會卡住...

![ubuntu install with mini iso hangs](https://i.imgur.com/FpWsjsO.png)

卡在這邊幾個小時了，都不會動

我在猜可能是mirror site 有問題，所以抓套件抓不到就卡住了？

一直卡著也不是辦法，於是又去ubuntu官網看了一下，發現有另外一個server的 ISO

這個叫 "Alternative Ubuntu Server installer"

在官網的這個位置 

https://www.ubuntu.com/download/alternative-downloads

![Alternative Ubuntu Server Installer](https://i.imgur.com/n0E1ea3.png)

進入後，會有個列表，找到 server amd64 的ISO，這個才是正確的

和第一次不同的是，這個沒有"live" ，很重要！

![Ubuntu alternative-downloads](https://i.imgur.com/c4GTujY.png)

用這個ISO開機，就可以正常的做出software raid，並且指定安裝作業系統，也不會有卡住的狀況

做出來的系統磁區大概是這樣
![ubuntu root on software raid](https://i.imgur.com/dyWIH7E.png)

這台VM的硬碟是透過10G網卡連到一個四塊Sandisk 240G SSD 組成的raid0空間

順便看一下速度
![10g nfs storage performance](https://i.imgur.com/V9WwIOC.png)

10G就是快！就是爽！


爽完之後，還是要確認一下... 首先先執行 sudo dpkg-reconfigure grub-pc

看看是不是兩顆硬碟都有裝 grub ，這樣萬一有一顆硬碟故障，另一顆才能啟動

![dpkg-reconfigure grub-pc](https://i.imgur.com/7xAcCbz.png)

看來因為是在安裝過程中，就指定了要把系統裝在raid上，所以ubuntu很聰明的，也自動把grub裝在兩顆硬碟上了

來試試看拔掉一顆硬碟還能不能正常運作

直接在proxmox 管理界面中，detach 一顆硬碟

![detach one of mirror raid](https://i.imgur.com/lLFcdk0.png)

果然報錯誤了

![mirror raid failed](https://i.imgur.com/1SFdVA0.png)

重開機看看，也沒有問題，可以順利開機！

開機過程有看到raid 只剩下一顆在運作的訊息

![mirror raid work with one disk only](https://i.imgur.com/oZNIN4D.png)

再來把硬碟加回去

然後用mdadm 指令加入分割區，raid就會開始rebuid了

![mdadm rebuild raid](https://i.imgur.com/3nu2Ij8.png)

所以，如果有打算要做software raid來安裝ubuntu 作業系統的，一開始就要選對ISO

才不會白忙那麼多時間啊！



