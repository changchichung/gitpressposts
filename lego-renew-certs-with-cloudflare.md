---
title: "lego 自動更新letsencrypt 全域憑證"
date: 2022-11-04T10:06:36+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-5.jp]
tags: ['lego','letsencrypt','ssl']
author: "Eric Chang"
keywords:
  - lego
  - letsencrypt
---
首先要確認 DNS 託管在 lego 有支援的DNS 廠商
可以到 github 去看，支援的廠商很多！
<!--more-->

我比較習慣cloudflare ，所以這邊講的是 cloudflare 的方式
首先登入 cloudflare 管理界面
接著點右上角倒三角，選擇 "My Profile"
![](https://i.imgur.com/08pxllD.png)

再來選擇左邊的 API Tokens , 然後點下面的Glocal API key 旁邊的 view 按鈕，來檢視你的 API KEY ，如果這邊沒有Global API Key  , 那就新增一個
![](https://i.imgur.com/FvOdhbJ.png)

之前的做法都是說在中間的API Token 去新增一組key 然後要選擇一些zone/dns 的權限，經過測試，確認這樣做沒辦法運作，必須要用 Global API Key 才行！

取得API Key  之後，剩下的就很簡單了
先去 https://github.com/go-acme/lego/releases 找到你對應的版本抓下來，我是習慣把檔案放在 /usr/local/bin/ 底下
又或者是可以用 apt 安裝

安裝完成之後，指令如下

第一次取得憑證
記得把 CLOUDFLARE_EMAIL 還有 CLOUDFLARE_API_KEY 改成你自己的
```
CLOUDFLARE_EMAIL=you@example.com CLOUDFLARE_API_KEY=abc123
lego --email you@example.com --dns cloudflare --domains *.example.org run
```

就會在home dir 底下的 .lego 目錄中，發現你的憑證檔案了，而且是全域通用的唷！(wildcard ssl certs)
![](https://i.imgur.com/6tZYCfy.png)

日後要更新憑證，指令也差不多，只是把最後一個 run 改成 renew 就可以了！

```
CLOUDFLARE_EMAIL=you@example.com CLOUDFLARE_API_KEY=abc123
lego --email you@example.com --dns cloudflare --domains *.example.org renew
```

是不是超級方便的啊！
