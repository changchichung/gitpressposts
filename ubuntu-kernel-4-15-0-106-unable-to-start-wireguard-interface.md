---
title: "Ubuntu 18.04 Kernel 4.15.0-106 Unable to Start Wireguard Interface"
date: 2020-06-22T09:05:42+08:00
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-16.jpg
tags: ['Ubuntu','Wireguard']
author: "Eric Chang"
keywords:
  - ubuntu
  - Wireguard
---

工作用的電腦，昨天終於難得的reboot了(uptime 看了一下，大概是三百多天)

結果重開機之後，發現原本在打tunnel 連 ptt 的 wireguard VPN 掛掉了

手動下指令也啟動不了

查了一下發現是 ubuntu 18.04 kernel 4.15.0-106 的包

看來就連kernel 最好都不要自動升級...

<!--more-->

一開始不管怎麼下指令要啟動wireguard Interface 都會出錯
```
root@hqdc034:~# wg-quick up wg0
[#] ip link add wg0 type wireguard
RTNETLINK answers: Operation not supported
Unable to access interface: Protocol not supported
[#] ip link delete dev wg0
Cannot find device "wg0"
root@hqdc034:~# wg-quick up wg1
[#] ip link add wg1 type wireguard
RTNETLINK answers: Operation not supported
Unable to access interface: Protocol not supported
[#] ip link delete dev wg1
Cannot find device "wg1"
```

因為很久沒動了，所以wireguard config 檔案應該是沒有問題

不過還是檢查看看？

```
root@hqdc034:~# wg showconf wg0
Unable to access interface: Protocol not supported
```

很好，果然不是config 的問題，看來是wireguard 某些套件有狀況了

用modprobe 檢查一下

```
root@hqdc034:~# modprobe wireguard
modprobe: FATAL: Module wireguard not found in directory /lib/modules/4.15.0-106-generic
```

OK ，找到問題了，看起來是新版本的kernel 有某些狀況？

anyway

要解決其實很簡單，要不就直接上 20.04  XD

要不就重裝 wireguard-dkms (這個似乎是新釋出的套件，本來沒有這個的)

```
root@hqdc034:~# apt-get install wireguard-dkms wireguard-tools linux-headers-$(uname -r)
正在讀取套件清單... 完成
正在重建相依關係          
正在讀取狀態資料... 完成
linux-headers-4.15.0-106-generic 已是最新版本 (4.15.0-106.107)。
linux-headers-4.15.0-106-generic 被設定為手動安裝。
wireguard-tools 已是最新版本 (1.0.20200513-1~18.04)。
wireguard-tools 被設定為手動安裝。
以下套件為自動安裝，並且已經無用：
  dmeventd liblvm2app2.2 liblvm2cmd2.02 libopts25 libreadline5 python-egenix-mxdatetime python-egenix-mxtools python-psutil
  python-psycopg2 python3-flask-htmlmin python3-htmlmin sntp
使用 'apt autoremove' 將之移除。
下列套件將會被升級：
  wireguard-dkms
升級 1 個，新安裝 0 個，移除 0 個，有 123 個未被升級。
需要下載 254 kB 的套件檔。
此操作完成之後，會多佔用 1,024 B 的磁碟空間。
是否繼續進行 [Y/n]？ [Y/n] y
下載:1 http://ppa.launchpad.net/wireguard/wireguard/ubuntu bionic/main amd64 wireguard-dkms all 1.0.20200611-0ppa1~18.04 [254 kB]
取得 254 kB 用了 1秒 (184 kB/s)          
（讀取資料庫 ... 目前共安裝了 298316 個檔案和目錄。）
準備解開 .../wireguard-dkms_1.0.20200611-0ppa1~18.04_all.deb ...

-------- Uninstall Beginning --------
Module:  wireguard
Version: 1.0.20200426
Kernel:  4.15.0-101-generic (x86_64)
-------------------------------------

Status: Before uninstall, this module version was ACTIVE on this kernel.

wireguard.ko:
 - Uninstallation
   - Deleting from: /lib/modules/4.15.0-101-generic/updates/dkms/
 - Original module
   - No original module was found for this module on this kernel.
   - Use the dkms install command to reinstall any previous module version.

depmod....

DKMS: uninstall completed.

------------------------------
Deleting module version: 1.0.20200426
completely from the DKMS tree.
------------------------------
Done.
Unpacking wireguard-dkms (1.0.20200611-0ppa1~18.04) over (1.0.20200426-0ppa1~18.04) ...
設定 wireguard-dkms (1.0.20200611-0ppa1~18.04) ...
Loading new wireguard-1.0.20200611 DKMS files...
Building for 4.15.0-106-generic
Building initial module for 4.15.0-106-generic
Done.

wireguard:
Running module version sanity check.
 - Original module
   - No original module exists within this kernel
 - Installation
   - Installing to /lib/modules/4.15.0-106-generic/updates/dkms/

depmod...


DKMS: install completed.
Updating loolwsd systemplate

```

跑完之後重起wireguard interface 就 OK 了

```
root@hqdc034:~# wg-quick up wg0
[#] ip link add wg0 type wireguard
[#] wg setconf wg0 /dev/fd/63
[#] ip -4 address add 192.168.10.2/24 dev wg0
[#] ip link set mtu 1420 up dev wg0
[#] resolvconf -a tun.wg0 -m 0 -x
[#] ip -4 route add 140.112.0.0/16 dev wg0
[#] ip -4 route add 104.31.0.0/16 dev wg0


root@hqdc034:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:10:18:14:0f:0a brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.34/24 brd 192.168.0.255 scope global enp3s0
       valid_lft forever preferred_lft forever
    inet6 fe80::2537:5b36:df2:7c0e/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s31f6: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 70:4d:7b:a3:66:f1 brd ff:ff:ff:ff:ff:ff

30: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none 
    inet 192.168.10.2/24 scope global wg0
       valid_lft forever preferred_lft forever
	   
	   
root@hqdc034:~# ip r
default via 192.168.11.253 dev enp3s0 src 192.168.11.34 metric 202 
10.25.0.0/16 dev LoyaltyNet proto kernel scope link src 10.25.25.1 linkdown 
104.31.0.0/16 dev wg0 scope link 
140.112.0.0/16 dev wg0 scope link 
192.168.10.0/24 dev wg0 proto kernel scope link src 192.168.10.2 
192.168.11.0/24 dev enp3s0 proto kernel scope link src 192.168.11.34 metric 202 
root@hqdc034:~# 

```



