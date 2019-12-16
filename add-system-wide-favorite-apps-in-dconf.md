---
title: "在ubuntu 18.04中，透過 dconf 設定系統層級的「我的最愛」/ Add System Wide Favorite Apps in dock with Dconf in ubuntu 18.04"
date: 2019-12-16T13:59:26+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-2.jpg
tags: ['ubuntu','dconf']
author: "Eric Chang"
keywords:
  - ubuntu
  - dconf
  - favorite-app
---

這幾天在ansible 寫了一份新的playbook給developer 用

然後user反映說，希望能在ubuntu 18.04 內建的dock 裏面新增一個gnome-terminal的icon

我才發現原來之前的寫法不能用在 ubuntu 18.04 上

只好又弄了一份出來

<!--more-->

有些task 我是直接從原本給14.04 client 用的直接套用過來

也沒有去特別注意

今天重新檢查才發現，舊的寫法是給 14.04 的unity用

但是 18.04 已經不用 unity 了，所以在設定dock這個task 雖然有成功，但是沒做用
(手術成功，但病人掛了？)

原來的寫法是在 /usr/share/glib-2.0/schemas/ 底下新增一個設定檔

然後用dconf 去產生設定

原本的內容長這樣
```
[com.canonical.Unity.Launcher]                                                                                                
favorites=['application://ubiquity.desktop', 'application://launchers.desktop', 'application://nautilus.desktop', 'application
://gnome-terminal.desktop', 'application://google-chrome.desktop', 'application://goldendict.desktop', 'application://stardict
.desktop', 'application://libreoffice-writer.desktop', 'application://libreoffice-calc.desktop', 'unity://running-apps', 'unit
y://expo-icon', 'unity://devices']
```

就如同前面所說，因爲18.04捨棄了 unity，所以這個config 等於沒有用了

新的步驟比較麻煩一點點

大概是

1. mkdir -p /etc/dconf/profile
2. vim /etc/dconf/profile/user
```
#This line allows the user to change the default favorites later.
user-db:user
#This line defines a system database named msb
system-db:msb
```
3. mkdir -p /etc/dconf/db/msb.d
4. vim /etc/dconf/db/msb.d/00_msb_settings
```
# Define default favorite apps
[org/gnome/shell]
favorite-apps = ['chromium-browser.desktop', 'firefox.desktop', 'gnome-terminal.desktop', 'nautilus.desktop']
```

把這些步驟改成 ansible 語法，再派送到client ，重開機之後就可以正確顯示設定好的「我的最愛」了




