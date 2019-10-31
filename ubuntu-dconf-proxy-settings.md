---
title: "[筆記] ubuntu 18.04 透過 dconf 修改系統 proxy / modify system proxy with dconf in ubuntu 18.04"
date: 2019-10-31T11:08:54+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-7.jpg
tags: ['dconf']
author: "Eric Chang"
keywords:
  - docnf
  - ubuntu
  - proxy
---

最近在準備升級client 的作業系統，從 ubuntu 14.04 準備升級到 18.04 或明年的 20.04

因為公司政策的關係，所以現在要連接internet ，需要申請

然後 user 再去系統的proxy 設定新增一個 PAC 檔

但是這個動作其實是去叫NetworkManager 這個服務

可是在18.04 上，我會把這個服務關掉，因為他會干擾我的DNS設定

所以想試試看有沒有辦法不使用 NetworkManager 服務

又能夠在 user level 修改 proxy 參數

就想到了用 dconf 來做

<!--more-->

dconf 是在 ubuntu 底下很好用的工具

可以用來觀察、修改使用者層級(user level)的系統設定\

不過有一些語法要注意

簡單說一下用法

觀察user level 系統變數的變化

開啟terminal 輸入以下指令

```
dconf watch /
```

這個可以觀察user到底修改了些什麼

只要是透過右上角的系統設定修改的值

這個指令都可以觀察到，非常好用

當找到了要修改的 KEY

就可以用

```
dconf read/write KEY
```

比如說我要修改proxy

我先用 dconf watch / 抓到了KEY是 /system/proxy/host 

那我就可以用 
```
dconf write /system/proxy/http/host "'192.168.1.7'"
dconf write /system/proxy/http/port '3128'
```
來把系統的http proxy 改成 192.168.1.7:3128

要注意的是，上面的 host 是字串，要用``` "''" ```包起來

下面的只是數字，就不用外面的``` "" ``` 了

不過這修改好像還是必須要NetworkManager 生效才行

還需要再測試看看



