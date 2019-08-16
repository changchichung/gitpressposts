---
title: "Bookstack Docker"
date: 2018-11-06T14:57:14+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-12.jpg"
categories: ['筆記']
tags: ['docker','Bookstack']
author: "Eric Chang"
---

Bookstack 是一套非常好用的線上"筆記"系統

他用圖書館/書本的概念，讓使用者可以建立自己的"圖書館"

同時在圖書館內建立不同的"書籍"

而且支援 Markdown 語法

其他的方式像是在nextcloud上編輯 md檔案(字體太小)

或者是boostnote(只能在本機)

都或多或少有點小缺點

Bookstack則是沒有這些問題，不過就是系統「大」了點...

不過還好有人做成docker的方式來啟動，大大的降低了建置的難度(其實也沒有很難啦，只是要裝個PHP、弄個DB而已)


<!--more-->

這個是專案的名稱 
#### solidnerd/docker-bookstack

gihub上的連結

https://github.com/solidnerd/docker-bookstack


因為都轉成docker了，所以安裝很簡單
先git clone回來
```
git clone https://github.com/solidnerd/docker-bookstack
```
然後依照他的說明，建立一個docker-compose.yml檔案，再視情況修改

底下是我的docker-compose.yml內容

```
 version: '2'                                                                                                                                         
 services:
   mysql:
     image: mysql:5.7.21
     environment:
     - MYSQL_ROOT_PASSWORD=secret
     - MYSQL_DATABASE=bookstack
     - MYSQL_USER=bookstack
     - MYSQL_PASSWORD=secret
     volumes:
     - mysql-data:/var/lib/mysql
  
   bookstack:
     image: solidnerd/bookstack:0.24.1
     depends_on:
     - mysql
     environment:
     - DB_HOST=mysql:3306
     - DB_DATABASE=bookstack
     - DB_USERNAME=bookstack
     - DB_PASSWORD=secret
     volumes:
     - uploads:/var/www/bookstack/public/uploads
     - storage-uploads:/var/www/bookstack/public/storage
     ports:
       - "0.0.0.0:8003:80"
  
 volumes:
  mysql-data:
  uploads:
  storage-uploads:
```

原則上我沒有修改什麼設定，先確認一下現在的bookstack版本 0.24.1 是目前最新的了

然後把 port 改成8003 ，避免去強碰 80 port

如果前面搭配 caddy 之類的反向代理，那不用去記 port 也沒關係。

好了之後，就執行 docker-compose up -d

然後 docker ps -a 看一下執行狀況
```
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS                         PORTS               NAMES
6b3333eabf30        solidnerd/bookstack:0.24.1   "/docker-entrypoint.…"   4 hours ago         Exited (0) About an hour ago                       docker-bookstack_bookstack_1
b8d74048eba1        mysql:5.7.21                 "docker-entrypoint.s…"   4 hours ago         Exited (0) About an hour ago                       docker-bookstack_mysql_1
```
應該可以順利跑起來

![bookstack運作畫面](https://i.imgur.com/NIUCJhN.png)

沒啥難度，簡單作一下紀錄

後面再來看看能不能在一個地方新增 md 檔案，然後可以自動傳到 hexo/hugo/ghost 的目錄，接著自動生成靜態檔案出來...

