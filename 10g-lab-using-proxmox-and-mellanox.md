---
title: "[筆記] 用 proxmox & Mellanox SFP 網卡土炮 10G LAB "
date: 2018-11-30T16:05:14+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-03.jpg"
categories: ['筆記']
tags: ['10G','筆記','mellanox']
author: "Eric Chang"
---

想做一個 10G 的 LAB 環境出來已經很久了。

只是礙於10G RJ45的卡太貴了，然後光纖的種類又太複雜

如果直接在淘寶購買，很怕會買錯(什麼LC/FC LC/LC 多模單模 單芯雙芯 SFP/SFP+ 又是什麼光模塊的一大堆規格)

所以一直沒有付諸行動。

硬體的工作很久沒碰了，剛好在蝦皮看到有個賣家在賣 mellanox 的X2網卡，以在台灣的價格來說，算很便宜的 (550)

聊了一下，跟他請教了關於線材、光纖模塊的問題，回答也都很快很到位

就直接下訂了兩張網卡、兩個光纖模塊、一條LC/LC 光纖線

就是到貨有點久，等了兩個禮拜左右，一直到昨天東西才寄到

今天就花了點時間測試一下


<!--more-->

先上個圖！

![10G LAB](https://i.imgur.com/34MNFkW.png)

簡單說，就是有兩台機器，分別安裝 proxmox (一台是新裝的，另一台是本來就在線上的LAB用機器)以及光纖網卡

Mellanox 這張 X2 的卡， proxmox 5.1 / 5.2 可以直接抓到，所以不必另外安裝驅動程式

硬體安裝很順利，不過軟體的設定就碰到點麻煩了，所以才想說作個筆記..

### <strong><font color="red">必須作 vmbridge 才能指定這個網卡給VM用 </font></strong>


安裝好網卡，開機，透過proxmox的WEB界面設定好網卡的資料後，原本以為可以直接使用了

但是proxmox 會提示需要重新開機才能變更設定

可是重新開機後，我兩台怎麼都ping不到對方

在這之前，我已經用兩台 ubuntu 18.04 client 測試過了，只要設定好IP就可以直接通

所以在這邊碰到這個問題，我滿訝異的

可是看網卡的燈號，明明就有亮起來，應該是正常的呀


原來，在proxmox 中，新增了網卡，並不是直接就可以拿來用

要先設定好 bridge ，然後才能起新的VM、指定新設定的 vmbridge 給這個新起的機器使用


### <strong><font color="red">Disk Cache type 要改</font></strong>


設定了新的 vmbridge 之後，就可以在新VM的設定畫面中，指定網卡走這個界面出去

可是這樣做出來的VM ，一直無法開機

錯誤訊息如下

```
kvm: -drive file=/zp/images/100/vm-100-disk-1.qcow2,if=none,id=drive-virtio0,format=qcow2,cache=none,aio=native,detect-zeroes=on: file system may not support O_DIRECT
kvm: -drive file=/zp/images/100/vm-100-disk-1.qcow2,if=none,id=drive-virtio0,format=qcow2,cache=none,aio=native,detect-zeroes=on: Could not open '/zp/images/100/vm-100-disk-1.qcow2': Invalid argument
TASK ERROR: start failed: command '/usr/bin/kvm -id 100 -name 123123 -chardev 'socket,id=qmp,path=/var/run/qemu-server/100.qmp,server,nowait' -mon 'chardev=qmp,mode=control' -pidfile /var/run/qemu-server/100.pid -daemonize -smbios 'type=1,uuid=da27a9ea-fd55-4542-b2a7-8d5b09bf7611' -smp '2,sockets=1,cores=2,maxcpus=2' -nodefaults -boot 'menu=on,strict=on,reboot-timeout=1000,splash=/usr/share/qemu-server/bootsplash.jpg' -vga std -vnc unix:/var/run/qemu-server/100.vnc,x509,password -cpu kvm64,+lahf_lm,+sep,+kvm_pv_unhalt,+kvm_pv_eoi,enforce -m 2048 -device 'pci-bridge,id=pci.2,chassis_nr=2,bus=pci.0,addr=0x1f' -device 'pci-bridge,id=pci.1,chassis_nr=1,bus=pci.0,addr=0x1e' -device 'piix3-usb-uhci,id=uhci,bus=pci.0,addr=0x1.0x2' -device 'usb-tablet,id=tablet,bus=uhci.0,port=1' -device 'virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x3' -iscsi 'initiator-name=iqn.1993-08.org.debian:01:b972d1ad783' -drive 'file=/zp/template/iso/ubuntu-18.04.1-live-server-amd64.iso,if=none,id=drive-ide2,media=cdrom,aio=threads' -device 'ide-cd,bus=ide.1,unit=0,drive=drive-ide2,id=ide2,bootindex=200' -drive 'file=/zp/images/100/vm-100-disk-1.qcow2,if=none,id=drive-virtio0,format=qcow2,cache=none,aio=native,detect-zeroes=on' -device 'virtio-blk-pci,drive=drive-virtio0,id=virtio0,bus=pci.0,addr=0xa,bootindex=100' -netdev 'type=tap,id=net0,ifname=tap100i0,script=/var/lib/qemu-server/pve-bridge,downscript=/var/lib/qemu-server/pve-bridgedown,vhost=on' -device 'virtio-net-pci,mac=A2:EA:45:EE:17:25,netdev=net0,bus=pci.0,addr=0x12,id=net0,bootindex=300'' failed: exit code 1
```

當然先去拜google，果然就看到了提示，需要把 磁碟的 Cache 從預設的 Default(No Cache) 改成 write through

![proxmox dish cache mode](https://i.imgur.com/cmClmGd.png)

為什麼？我也不知道..不知道是不是因為我把磁碟種類選成用 Virtio Block 的關係

總之呢，改完之後就可以了 ...

### <strong><font color="red">必須手動設定路由</font></strong> 

#### Update

```
這邊可能是我有誤解，應該不需要先在pve 本機設定 10G網卡的 IP
直接進web console 去設定 vmbr1 就好了
```

設定好新的VM，開機、設定IP、重開機之後，會發現還是ping 不到另一台機器..(翻桌！)

只好又去拜google ，就看到了底下這篇

https://forum.proxmox.com/threads/how-to-add-second-nic.40905/

大概點出了方向，必須要手動增加路由(感覺有點蠢)

像我的光纖網卡走的是 192.168.50.0/24 ，就要去把原有的192.168.50.0/24的路由給砍掉，然後再新增(是不是很蠢？)

```
root@ssd:/etc/network# ip route del 192.168.50.0/24
root@ssd:/etc/network# ip route add 192.168.50.0/24 dev vmbr1
root@ssd:/etc/network# ip route

default via 192.168.11.253 dev vmbr0 onlink 
192.168.11.0/24 dev vmbr0 proto kernel scope link src 192.168.11.215 
192.168.50.0/24 dev vmbr1 scope link 
root@ssd:/etc/network# 

```
OK ping 一下對面看能不能過
```
root@ssd:/etc/network# ping 192.168.50.10
PING 192.168.50.10 (192.168.50.10) 56(84) bytes of data.
64 bytes from 192.168.50.10: icmp_seq=1 ttl=64 time=0.083 ms
64 bytes from 192.168.50.10: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 192.168.50.10: icmp_seq=3 ttl=64 time=0.080 ms
64 bytes from 192.168.50.10: icmp_seq=4 ttl=64 time=0.126 ms
^C
--- 192.168.50.10 ping statistics ---
```

GOOD ！很好！通了一邊，另外一邊就照辦，兩邊都通了，就可以開始來測試速度了

p.s 這個路由不知道需不需要每次都手動增加，或者是有哪個config可以在開機時載入

沒記錯的話，應該是在 /etc/network/if-up.d/ 新增一個 route 檔案

不過這部份我不是很確定就是了

所以自己寫了一個 script 來用..


### <strong>iperf 測試速度</strong>


在linux 上，我習慣用 iperf 來測試兩台主機的連接速度

兩邊都用 apt install iperf 裝好套件

然後找一台作為 server ，執行
```
iperf -s
```

然後到另一台，去執行
```
2018-11-30 15:36:58 [minion@ubuntu ~]$ iperf -d -t 600 -P 10 -c 192.168.50.200
WARNING: option -d is not valid for server mode
------------------------------------------------------------
Client connecting to 192.168.50.200, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
[  3] local 192.168.50.199 port 40980 connected with 192.168.50.200 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-600.0 sec   641 GBytes  9.18 Gbits/sec

```
哈哈哈，有目有！測試速度來到了 9.18 Gbits 啊！ 就是一個爽啊！

記得那個 server IP 是你 VM 裡面設定的 IP，不是 proxmox 上面的

同場加映走 1Gb 網路的測試結果
```
2018-11-30 16:39:37 [minion@ubuntu ~]$ iperf -d -t 600 -P 10 -c 192.168.11.171
WARNING: option -d is not valid for server mode
------------------------------------------------------------
Client connecting to 192.168.11.171, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
[  3] local 192.168.11.55 port 38582 connected with 192.168.11.171 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-600.0 sec  65.8 GBytes   941 Mbits/sec

```

192.168.11.171 跟 192.168.50.200 是同一台機器，只是一個是10G網卡，一個是onboard的 1Gb 網卡

速度果然是提高了十倍呀，果然就是一個爽啊！！


### <strong>實際開VM來測試看看</strong> 

~~上面的測試，是兩台PVE HOST之間的連線測試~~

接下來，要實際測試在PVE中，建立新的VM，一台安裝FreeNAS 作為storage，另一台則是一般的client

步驟簡單來說，就是在ssd 這台PVE 建立一個新的VM，然後安裝FREENAS，並且提供NFS/iscsi 給另一台PVE Host作為storage來源

新增storage選NFS，填入必要資訊後，在這台主機上，建立一個新的VM，磁碟選擇剛剛連接的NFS

##### <strong>要特別注意，freenas的NFS Share的參數要改</strong>#####

在 mapuser/mapgroup這邊要改成 root/wheel 不然會有無法寫入的問題

![FREENAS NFS Sharing](https://i.imgur.com/Yn7qYdK.png)

安裝完之後，實際跑一下 dd 看看速度多少

```
Last login: Mon Dec  3 03:10:54 2018
2018-12-03 03:15:03 [administrator@ubuntu ~]$ dd if=/dev/zero of=testfile bs=10240 count=1000000
1000000+0 records in
1000000+0 records out
10240000000 bytes (10 GB, 9.5 GiB) copied, 9.63458 s, 1.1 GB/s
```

```
2018-12-03 03:17:28 [administrator@ubuntu ~]$ dd if=/dev/zero of=testfile bs=20480 count=1000000
1000000+0 records in
1000000+0 records out
20480000000 bytes (20 GB, 19 GiB) copied, 16.0786 s, 1.3 GB/s
```

```
2018-12-03 03:17:50 [administrator@ubuntu ~]$ dd if=/dev/zero of=testfile bs=4096 count=1000000
1000000+0 records in
1000000+0 records out
4096000000 bytes (4.1 GB, 3.8 GiB) copied, 4.80629 s, 852 MB/s
2018-12-03 03:25:23 [administrator@ubuntu ~]$ 
```
可以看到不但大檔案速度都很快，就連小檔案(4096)居然也有852MB

我底層也不過就是四顆 SATA3 sandisk 240G SSD 而已啊

如果都換成PCI-E SSD ，嘿嘿...(流口水

- - - 

不過呢，這個也只是自己建的LAB玩玩看而已

真的要放到 production 環境去，我也還沒啥把握 (畢竟都是中古、二手、退役的產品拼湊起來的)

而且沒有10G Switch ，所以只能點對點連接

說不定等到對岸的 10G Switch 開始大降價 (我覺得 8 port SFP+ / NTD $2000 左右我應該就會出手了)

再來把10G 的環境弄完整一點！








