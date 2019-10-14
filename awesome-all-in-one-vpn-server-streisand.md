---
title: "[筆記] 超強的ALL-in-One VPN Server streisand / Awesome All in One Vpn Server Streisand"
date: 2019-10-14T13:59:58+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['vpn','wireguard']
author: "Eric Chang"
keywords:
---

最近上班閒得發慌，沒事就上 github 找看看有沒有什麼好玩的專案

就不小心發現了這個 streisand

https://github.com/StreisandEffect/streisand

玩了一下，發現這根本就是終極的VPN Server solution ..

<!--more-->

包含了各種常見的VPN 套件，像是

* OpenConnect / Cisco AnyConnect
* OpenVPN
* Shadowsocks
* WireGuard

並且直接整合了幾間比較大的雲端主機服務商


* Amazon Web Services (AWS)
* Microsoft Azure
* Digital Ocean
* Google Compute Engine (GCE)
* Linode
* Rackspace

透過指令，可以簡單的在上述的空間建立一台新的 VPS (可惜沒有vultr)

安裝步驟什麼的就不說了，因為整合了 ansible 所以過程很簡單

而且在安裝完成後，會自動生成相關文件

文件中包含了怎麼連線的方式，以及連線設定檔，甚至連 QRCODE 都幫你做好

真的是超級方便的VPN工具！

