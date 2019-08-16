---
title: "[筆記] 詭異的client&server間連線的問題，或許跟KVM有關係？"
date: 2018-11-08T18:01:23+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-4.jpg"
categories: ['筆記']
tags: ['ubuntu','筆記']
author: "Eric Chang"
---

這是發生在一個夜黑風高的寂寥深夜..... ( What The FXXX ... )

來到這個環境之後，有一個很詭異的狀況一直困擾著我

在每個分公司，都會有一台伺服器作為KVM Host

上面跑兩台VM，一台作為ansible controller (目前沒作用)

另一台作為這邊所謂的 "Build Server"

用途包含了DHCP Server / Proxy Server (squid3) / APT Proxy (squid-deb-proxy)

問題就發生在這台 Build Server 上...

<!--more-->

有陣子花了點時間去檢查各個分公司的網路環境，確保每一台Build Server都能夠連接Internet

然後找了一個離總部最近的據點，把這些電腦連接Internet 的方式改為用 proxy 來控制

在proxy內加入了 allowhost 的設定，然後把user電腦上的瀏覽器都代入 proxy server (firefox/chrome 的設定方式不同)

```
acl localnet src 192.168.28.0/24
acl allowhost src "/etc/squid3/allowhost.txt"
acl localdomain dstdomain "/etc/squid3/localdomain.txt"
acl SSL_ports port 443
acl Safe_ports port 80      # http                                                                                                                          
acl Safe_ports port 21      # ftp
acl Safe_ports port 443     # https
acl Safe_ports port 70      # gopher
acl Safe_ports port 210     # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280     # http-mgmt
acl Safe_ports port 488     # gss-http
acl Safe_ports port 591     # filemaker
acl Safe_ports port 777     # multiling http
acl CONNECT method CONNECT
```

一開始這樣作還相安無事，但是呢，慢慢的時不時會有USER反應說無法連接 Internet

照理來說，因為都是透過proxy上網，所以如果是proxy server出問題，那其他電腦應該也不行上Internet 

但如果這樣的話，那就一點也不詭異了呀(攤手)

實際上的狀況是，只有反應的USER的電腦無法連接Internet

然後真的詭異的來了

用USER電腦去 ping proxy server ，有時候會通，有時候不通..

從Proxy Server去 ping USER電腦，也是類似的狀況

可是我卻可以透過IPSEC VPN，分別SSH連接到這兩台機器上

這代表兩台的網路都OK呀..

正當我百思不得其解的時候，突然 USER電腦那邊的 ping 有反應了

變成可以 ping proxy Server 了！ (What the FXXX !!!!)

我什麼都沒改呀...

update: 2018/11/19

剛剛在測試一台機器，又發生這個問題

兩台都ping不到對方

![ping不到](https://i.imgur.com/gSD086o.png)


什麼事也沒做，就是把ping中斷，然後再ping 一次，居然就可以了


![又ping到了](https://i.imgur.com/rvtw0hh.png)


##真他X的詭異啊！

<hr>

反正呢...

這種狀況三不五時就會出現一次，會出現在哪一台電腦也不一定

不過，依照觀察到的狀況來說，似乎都是發生在很少開機的電腦上

然後呢，因為底層是 KVM

我也嘗試過用virsh 去restart VM 或者是 restart network

有時候可以解決，有時候又還是不能連接

於是另外測試安裝了 proxmox VE 的虛擬平台

在上面起一台新的Server，再用 ansible 做成 build server的角色

這樣子作的機器，就不會發生這種狀況

所以我在猜是不是跟底層是KVM有關係..

不過要動這個的話，工程有點大，手邊也沒那麼多機器可以替換(很慘)

暫時先保留這個作法，等到下次再發生這狀況

再來找老闆看這情形，然後來討論要不要換掉各分公司的VM Host...



