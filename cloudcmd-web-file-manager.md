---
title: "[筆記] WEB 檔案管理 Cloudcmd Web File Manager"
date: 2021-07-20T09:19:47+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-2.jpg
tags: ['file manager','cloudcmd']
author: "Eric Chang"
keywords:
  - cloudcmd
  - filemanager
---

最近又接到之前處理過的需求，要讓使用者可以在外部上傳、編輯 yaml 檔案

之前是用 gohttpd 來做

可是不支援線上編輯 yaml 檔案
<!--more-->

這次找到了 cloudcmd

https://github.com/coderaiser/cloudcmd

簡單好用、不需要太多設定，但是想要的設定大致上也都有、有提供docker-compose

同時也支援多種檔案的預覽、編輯功能

算是很不錯的一個web-based 的檔案管理系統



### 登入時，會詢問帳號密碼

也可以設定成不詢問直接進入

![login](https://i.imgur.com/29NVQ66.png "login_pass")



### 支援多種檔案的預覽和編輯

MP4 影片

![previer1](https://i.imgur.com/9adI50o.png "preview1")



JPG檔案

![preview_2](https://i.imgur.com/ZHQMgI7.png "preview2")



CSV 檔案

![20210720091231-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210720091231-image.png)



編輯YAML

![20210720091330-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210720091330-image.png)



空白處按右鍵的功能表

![20210720091445-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210720091445-image.png)



檔案功能表

![20210720091514-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210720091514-image.png)



系統功能設定

![20210720091635-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210720091635-image.png)


目前用起來感覺還不錯，應該會推薦這套上去給老闆決定要不要開給user使用！

