---
title: "[筆記] 解決無法建立zpool 的錯誤 / Fix Zpool Device Busy Using dmsetup"
date: 2019-04-01T15:56:27+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['zfs']
author: "Eric Chang"
---

今天把其中一台proxmox 加上10G 光纖網卡，準備和另一台proxmox 組成10G 環境進行測試

想說把本機的zpool 拆掉，重新建立一個raid0 的空間來做clone/migrate

可是一直出現device busy的錯誤訊息

<!--more-->

```
root@pve:~# zpool create zp sdb1
cannot open '/dev/sdb1': Device or resource busy

```
可是我沒有mount 這個分割進來

而且我也可以用fdisk 去切sdb ，代表sdb 沒有真的被使用

找了很久，終於找到這個dmsetup指令

先用 ```dmsetup info -C ``` 來看現在的狀態

```
root@pve:~# dmsetup info -C
Name                              Maj Min Stat Open Targ Event  UUID                                                                
ST2000DM001-1ER164_W4Z3KKJB       253   3 L--w    0    1      0 mpath-ST2000DM001-1ER164_W4Z3KKJB                                   
pve-swap                          253   0 L--w    2    1      0 LVM-6Hle5UGjtr8NQQsbMrYlSdGXZklwAi87Kq9NlzQa6xvgiHOEP3Ekx72i5yYNaupf
pve-root                          253   1 L--w    1    1      0 LVM-6Hle5UGjtr8NQQsbMrYlSdGXZklwAi87geZeFZsQsgYUbI1ZJU4lKD86TVd1MNrq
ST2000DM001-1ER164_W4Z3KM2F-part1 253   5 L--w    0    1      0 part1-mpath-ST2000DM001-1ER164_W4Z3KM2F                             
ST2000DM001-1ER164_W4Z3KM2F       253   2 L--w    1    1      0 mpath-ST2000DM001-1ER164_W4Z3KM2F                                   
ST2000DM001-1ER164_W4Z3GYNJ       253   4 L--w    0    1      0 mpath-ST2000DM001-1ER164_W4Z3GYNJ                                   
```

除了那兩個LVM開頭的以外，其他都不應該出現在這裡才對
移除掉應該就可以了
要照順序，像那個有part1 的，就要先移掉，才能移掉底層

```
root@pve:~# dmsetup remove ST2000DM001-1ER164_W4Z3KM2F-part1
root@pve:~# dmsetup remove ST2000DM001-1ER164_W4Z3KM2F
root@pve:~# dmsetup remove ST2000DM001-1ER164_W4Z3KKJB
root@pve:~# dmsetup remove ST2000DM001-1ER164_W4Z3KM2F
root@pve:~# dmsetup remove ST2000DM001-1ER164_W4Z3GYNJ
```

再來建立zpool 就 OK了
```
root@pve:~# zpool create zp sdb sdc sdd
root@pve:~# zpool status
  pool: zp
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	zp          ONLINE       0     0     0
	  sdb       ONLINE       0     0     0
	  sdc       ONLINE       0     0     0
	  sdd       ONLINE       0     0     0

errors: No known data errors
root@pve:~# 
```

