---
title: "[筆記] 在gcp 中用wireguard建立VPN時，不要用 10.0.0.0/16 網段/Do No Use 10 0 0 0 Private Ipaddr in GCP"
date: 2019-08-16T10:18:06+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-8.jpg"
categories: ['筆記']
tags: ['vpn','wireguard']
author: "Eric Chang"
---

最近一直在玩 wireguard ，先前把各個分公司和總部的VPN 改用 wireguard 建立

想說再打個VPN tunnel 來當跳板連 ptt 好了

因為wireguard 建立很簡單，而且又可以指定想要繞出去的路由，不會影響原本的網路環境

本來是在vultr 的VPS上面建立這個tunnel 

但是那台VPS連去ptt 很頓，卡卡的

所以改用google cloud platform 的free tier 來做

反正只是拿來當跳板，不會有什麼流量、運算產生，可以一直保持免費的狀態

<!--more-->

GCP的申請、設定就不多說了

這次碰到的怪異現象是當wireguard 都已經設定好，client 也都連上了之後

會發生client 開不了 www.google.com.tw / youtube / google map 等等google 服務的狀況

VPN確定是通的，我可以在client 這邊連上其他網站，但就是google的服務開不了

後來不知道是怎麼樣，突然靈機一動，因為一開始設定server/peer 都是用 10.0.0.x/24 的IP

想說會不會是因為這個也是google cloud platform 預設的LAN IP 網段，所以沒辦法繞出去

看一下設定，確認一下這個想法對不對，果然是這樣沒錯

![](https://i.imgur.com/XkrH4Pa.png)

解決方法很簡單，要不修改VPS的內部IP，要不修改wireguard的設定

當然我是選擇改wireguard ，因為簡單嘛！

修改後的configuration 長這樣
```
[Interface]
Address = 192.168.10.1/24
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o ens4 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o ens4 -j MASQUERADE
ListenPort = 12000
PrivateKey = E..........................E

#OFFICE DESKTOP
[Peer]
PublicKey = W...........................W
AllowedIPs = 192.168.10.2/32

#ANDROID
[Peer]
PublicKey = w............................w
AllowedIPs = 192.168.10.3/32

#HOME
[Peer]
PublicKey = 2.........................................2
AllowedIPs = 192.168.10.4/32
```

重起wireguard (或者說重起 wg0 這個interface)之後，client 開google 網頁就正常了

client 這邊，也是簡單設定一下，把要透過跳板出去的IP 改走wireguard 出去

底下這個，就是把往台大(140.112.0.0) 和 term.ptt.cc(104.31.0.0)的封包改走wireguard

```
[Interface] 
PrivateKey = e............................e
Address = 192.168.10.2/24 
DNS = 8.8.8.8 
MTU = 1420 

[Peer] 
PublicKey = q...........................q
Endpoint = public_ip_of_gcp:12000
AllowedIPs = 140.112.0.0/16,104.31.0.0/16,192.168.10.1/32
PersistentKeepalive = 25
```

然後看一下路由對不對

```
2019-08-16 10:34:21 [cch@hq34 ~]$ traceroute term.ptt.cc
traceroute to term.ptt.cc (104.31.231.9), 30 hops max, 60 byte packets
 1  192.168.10.1 (192.168.10.1)  191.826 ms  192.556 ms  192.678 ms
 2  * * *
 3  * * *
 4  * * *
 5  104.31.231.9 (104.31.231.9)  203.918 ms  203.982 ms  203.979 ms
2019-08-16 10:34:33 [cch@hq34 ~]$ 
```

果然是走wireguard (192.168.10.1) 出去 ，跳板成功！


