---
title: "Init Script in Openwrt to Start Leproxy/在openwrt 新增自動啟動leproxy的script"
date: 2021-09-29T14:38:10+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-8.jpg
tags: ['openwrt']
author: "Eric Chang"
keywords:
  - openwrt
---

最近在逐步的把舊有的VPN Router 汰換掉，改用wireguard 來作 full mesh site-to-site VPN

不過這是另外的故事了...

在把wireguard VPN 都搞定之後，才發現原來 openwrt 的 uhttpd 要加上 letsencrypt 的免費憑證有點難搞

網路上大部分都介紹用 acme.sh ，我是有測試出來啦

但是跟網路上的方法不太一樣了，新增了滿多步驟的，覺得很麻煩

想到向來愛用的 leproxy ，既然是 golang 開發的，又是open source

就拿來compile 給openwrt router 用用看

想不到還真的可以， golang 真是棒！

不過也還是要順手改一些openwrt 東西才行

還是簡單作個筆記好了

<!--more-->

#### compile leproxy for arm64

當然要先確認好自己的環境有沒有裝了golang 可以用來編譯，這部分就不多提了。


##### 下載並編譯 leproxy
```
git clone https://github.com/artyom/leproxy
cd leproxy
GOOS=linux GOARCH=arm64 go build .
mv leproxy leproxy.arm64
```

##### copy leproxy.arm64 to router
```
scp leproxy.arm64 root@192.168.0.254:/root/leproxy.arm64
```

#### 接著 ssh 登入 router 作相關設定

ssh root@192.168.0.254 

##### 建立/etc/leproxy/mapping.yml

```
mkdir -p /etc/leproxy
vim /etc/leproxy/mapping.yml
```

內容大概長這樣，一次可以不止一行
然後要注意 hqvpnrouter.abc.com 這個域名要先存在 A 記錄並指向這臺 router

```
hqvpnrouter.abc.com: 192.168.0.254:81
```

前面是這臺機器的hostname , leproxy 會用這個hostname 去申請免費的憑證
後面是要把hqvpnrouter.abc.com 的要求轉到哪裡？這邊就是轉到本機(192.168.0.254)的 81 port

##### 修改 uhttpd config

因為leproxy 會佔用 80 ,443 兩個port
所以要把 uhttpd 改去別的port 工作
順便把 https 的設定拿掉，讓leproxy 去煩惱

```
# HTTP listen addresses, multiple allowed
	list listen_http	0.0.0.0:81
	list listen_http	[::]:81

	# HTTPS listen addresses, multiple allowed
	#list listen_https	0.0.0.0:443
	#list listen_https	[::]:443

	# Redirect HTTP requests to HTTPS if possible
	option redirect_https	0
```

然後先重啟 uhttpd 

```
/etc/init.d/uhttpd restart
```

看看 uhttpd 是不是已經改到 port 81
```
[200~root@HQ_VPN_ROUTER:~# netstat -antlp
netstat: showing only processes with your user ID
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:81              0.0.0.0:*               LISTEN      1491/uhttpd
tcp        0      0 10.2.3.2:53             0.0.0.0:*               LISTEN      3540/dnsmasq
```

好，這時候就可以用以下指令來測試leproxy 是不是可以正常運作

cacheDir 是會被用來存放leproxy 取得的免費憑證，必須要先存在系統中
或者是要存放在 /tmp , /root 也都可以

```
/root/leproxy.arm64 -map /etc/leproxy/mapping.yml -email chchang@abc.com -cacheDir /etc/acme/
```

##### 修改 firewall config

加入底下這段

```
config redirect
	option dest_port '443'
	option src 'wan'
	option name 'https for leproxy'
	option src_dport '443'
	option target 'DNAT'
	option dest_ip '192.168.0.254'
	option dest 'lan'
	list proto 'tcp'
```

重啟 firewall

這時候應該可以用 https://vpnrouter.abc.com 的方式來開啟這臺router 的管理界面


#### 建立 init script

在 /etc/init.d 中新增一個檔案叫 leproxy 

內容如下
```
#!/bin/sh /etc/rc.common
# Example script
# Copyright (C) 2007 OpenWrt.org

START=99

start() {        
        echo "leproxy starting"
    	/root/leproxy.arm64 -map /etc/leproxy/mapping.yml -email chchang@abc.com -cacheDir /etc/acme/ > /dev/null 2>&1 &
        
}
stop () {
	echo "leproxy stopping"
	killall leproxy.arm64
	}
```

##### 改一下file permission

```
chmod u+rwx /etc/init.d/leproxy
```

##### 設定開機自動啟動

```
/etc/init.d/leproxy enable
```

##### 啟動leproxy
```
/etc/init.d/leproxy restart
```

開啟 https://vpnrouter.abc.com 再做一次確認

