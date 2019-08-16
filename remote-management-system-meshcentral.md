---
title: "linux底下遠端遙控&管理的好用系統 Meshcentral / Remote Management & control system Meshcentral"
date: 2019-06-20T11:03:12+08:00
draft: false 

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-5.jpg"
categories: ['筆記']
tags: ['linux','remote','meshcentral']
author: "Eric Chang"
---

之前在LAN/windows環境下，一直都是用ultravnc/winvnc/tigervnc之類的VNC軟體

但是如果要過 internet ，就會碰到各種開port的問題

在這種環境下，就有了當時 teamviewer 的橫空出世

解決了開PORT的問題，讓被控端(通常是資訊技術相對弱勢，需要接受幫助的一方)不需要懂太多

只要下載teamviewer被控端，開啟後報ID 給協助者就好了

<!--more-->

好景不常...ㄟ，好像也不能這麼說

teamviewer 是一套商業授權軟體，要買license的

不知道用什麼方式偵測，一開始可以用，但是後來會出現視窗警告，再來就不讓你用了。

於是又有了 anydesk 的出現

anydesk 的定義就是免費軟體，所以沒有授權問題

可是 anydesk 在初期有頗多狀況，比如像是windows的UAC，或者是畫面反應速度過慢

又或者是常常自動變成view only ，無法操作的狀況

當使用者需要讓我用anydesk連線進去時，通常就已經是電腦有些什麼狀況了

然後還要面對anydesk的種種問題，實在是讓人很抓狂！

終於某次因緣際會，讓我找到了這篇的主角

## Meshcentral ##

簡單介紹一下Meshcentral 的優點

### 安全性高 ###
  teamviewer & anydesk 之所以不需要開port，就是被控/遙控兩邊都是先連線到他們提供的伺服器(通常在國外)

  有人會擔心這樣畫面會不會被擷取之類的 我是認為想太多了啦，但就真的有人會擔心這種問題啊(嘆

  meshcentral則不同，他是安裝在LAN的機器上(當然，要跨WAN也可以，就開port吧)

  所以原則上從server到client這段的連線，都是在LAN中加密進行，不會有需要上傳到廠商伺服器的問題

  自然安全性就高了很多

### 速度快 ###
  
  如同前面所述，因為都在LAN中進行，不需要透過廠商在國外的Server，所以操作的速度很快！

### 操作簡單 ###

  一開始meshcentral 會需要在client 安裝agent，在比較早期一點的版本，這個動作需要先用anydesk連上被控端

  然後去開啟連結、以管理者權限進行安裝

  新版改善了這個問題，可以直接產生一個invite url ，直接發信給user ，請user去點連結進行安裝

  當然要先做好安裝步驟說明就是了，不然windows會判斷這個是有問題的檔案，不讓執行

  裝完之後，被控端就不會再看到這東西了(所以也無從關閉，除非從service.msc 去stop)

  而且也不會碰到煩人的UAC訊息跳出來時，畫面上的按鈕無法點選的問題(anydesk就有這種狀況，需要透過config去排除)

### 安裝容易 ###

  meshcentral的安裝很簡單，在ubuntu 18.04 server 的環境底下執行以下指令

  ```
  #install nodejs/npm
  sudo apt install nodejs npm 
  #make meshcentral working folder
  sudo mkdir -p /opt/meshcentral
  #install meshcentral
  cd /opt/meshcentral;npm install meshcentral
  # start meshcentral
  cd /opt/meshcentral/node_modules/meshcentral;node meshcentral --cert {{ servier_ip_address }}
  ```

  這樣就把meshcentral安裝&啟動了

  應該是可以改用systemd or supervisor 來控制了，那個是另外的部份了


- - -


### Meshcentral 簡易操作說明 ###

meshcentral 在操作上也很簡單，第一次安裝成功後，開啟meshcentral的頁面

首先建立一個管理者帳號，接著用管理者帳號進入後，建立一個group

然後進入這個group ，會看到一個 invite的連結

![](https://i.imgur.com/MNrv2Sw.png)

點擊invite之後，會詢問要產生多長有效時間的連結，如果很懶，就直接選unlimited

然後把這串URL記下來，發給所有user(最好自己先run過一遍，做一份安裝步驟一起發給使用者)

![](https://i.imgur.com/cXQBEWU.png)

等到user照說明，安裝agent之後，在管理界面上就會看到client出現了

![](https://i.imgur.com/bQL6JJE.png)

有亮起來的圖示就代表client online ，可以直接進去操作

操作畫面可以操考以下影片，我是透過internet(兩邊都是CHT雙向100) 去控制遠端電腦播放youtube

可以看到畫面是非常地流暢！

{{< youtube p6-BBYW51qc >}}


### 總結 ###

這麼好用的系統，還不快去裝一套起來！



