---
title: "[推薦] ulauncher ubuntu 18.04 底下，好用的 app launcher / Recommended Ulauncher in Ubuntu 1804"
date: 2019-10-04T14:12:15+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['']
tags: ['launcher']
author: "Eric Chang"
---

這兩天在找關於在 ubuntu 中做搜尋的軟體

意外找到一個非常好用的工具 ulauncher

<!--more-->

官方網站： https://ulauncher.io/

簡單的說，這東西可以讓你不需要ubuntu 的 dash 輸入關鍵字尋找 app

舉例來說

如果我想要啟動 libreoffice Calc

通常我會按一下鍵盤上的 windows/super 鍵，然後輸入calc

像是圖片中這樣

![](https://i.imgur.com/hlmtneN.png)

OK ，那這樣有什麼問題呢？

最大的問題應該是fcitx 的 bug ，在這邊不能輸入中文搜尋，如果只是要找應用程式還好

但是要找檔案就會有問題了

那用 ulauncher 有什麼不同呢？

首先，因為 ulauncher 內建的呼叫快捷鍵是 ctrl+space 

跟 fcitx (對，又是他) 切換中文的快捷鍵衝突

所以要改掉，可以改成自己喜歡的任意組合，像我就改成 ctl+ esc

叫出ulauncher 的視窗後，直接輸入 calc 也有一樣的效果

![](https://i.imgur.com/RTYBGeT.png)

當然啦，如果只是這樣，那也沒什麼了不起的

ulauncher 最大的好處是他支援各式各樣的extension

可以在網站上瀏覽這些擴充套件，最好選擇 V2.0 的，他新舊版本不相容

https://ext.ulauncher.io/

拿幾個我裝的 extension 來示範

比如我常寫 ansible playbook

可是又很常忘記某些模組、語法怎麼用

所以我都要開著瀏覽器去 ansible 官網看資料

現在我可以用 DevDocs 這個extension 直接查語法

像是這樣

![](https://i.imgur.com/O3q9rqG.png)

按下enter 或者 alt+1 就可以開啟相關語法的網頁

又或者是常常要去查 IP，也有相關的extension 可以用 

![](https://i.imgur.com/9Xmv3pn.png)

再來就是 unicode 字元查詢

可以很簡單的複製 unicode 裡面的特殊字元

像是狗貓牛豬

## 🐕🐈🐄🐖

房子車子孩子

## 🏠🚚👦

是不是很方便！(當然前提是平常有用到這些符號啦)

最後是我覺得最好用的搜尋檔案

之前在搜尋檔案的時候，都是先去點開檔案總管，然後用裡面的搜尋
(好啦，我承認其實我很少用這個功能，檔案沒有多到記不得放在哪裡)
(可是 👩 說很需要....)

不過這個功能要搭配 tracker 這個套件，我不確定 ubuntu 預設有沒有裝

裝好之後，做一些簡單的設定，就可以直接搜尋檔案名稱了

![](https://i.imgur.com/I5ZOY2m.png)

根本殺手級套件啊！

強烈推薦大家一定要裝起來玩玩看！




