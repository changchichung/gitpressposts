---
title: "[筆記] 在openwrt 中檢查網路埠的連接速度/ Check Port Speed in Openwrt"
date: 2020-07-15T10:35:01+08:00
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-10.jpg
tags: ['openwrt']
author: "Eric Chang"
keywords:
  - openwrt
---

最近在玩ansible + openwrt + wireguard

ansible 腳本寫好之後，可以把config 佈署到 openwrt 上

當然前提是最好用同樣的機器，不同的機器在config 上會有一些差異

但是這些差異常常就會造成無法連線、無法使用的狀況

BTW 我是用 ubiquiti 的 edgerouter X 來做

都弄好之後，就想說來跑個iperf3 測試一下連線速度

也好和之前做的 IPSEC 比較一下

結果很奇怪的是，明明一樣的機器、一樣用ansible 跑出來的config

但是有一台edgerouter X 的VPN 連接速度就是特別慢

而且速度都剛好卡在 99.X Mb 左右

就讓我很納悶了...


<!--more-->

於是想說來檢查一下網路孔的狀態

但是因為openwrt 精簡了很多指令，所以一些linux 上常用的指令都看不到實際的連線速度

後來終於找到這一篇

https://forum.openwrt.org/t/change-interface-br-lan-from-100-mb-to-1-gigabit-help-me/21914

其中有提到這個指令

```shell
swconfig dev switch0 show
```

所以在有問題的那台機器跑一次，結果就發現了port0 的連線速度只有100BaseT
```shell
root@OpenWrt-15:~# swconfig dev switch0 show
Global attributes:
	enable_vlan: 1
	mib: Switch MIB counters
PPE_AC_BCNT0: 0
PPE_AC_PCNT0: 0
PPE_AC_BCNT63: 0
PPE_AC_PCNT63: 0
PPE_MTR_CNT0: 0
PPE_MTR_CNT63: 0
GDM1_TX_GBCNT: 0
GDM1_TX_GPCNT: 0
GDM1_TX_SKIPCNT: 0
GDM1_TX_COLCNT: 0
GDM1_RX_GBCNT1: 0
GDM1_RX_GPCNT1: 0
GDM1_RX_OERCNT: 0
GDM1_RX_FERCNT: 0
GDM1_RX_SERCNT: 0
GDM1_RX_LERCNT: 0
GDM1_RX_CERCNT: 0
GDM1_RX_FCCNT: 0
GDM2_TX_GBCNT: 0
GDM2_TX_GPCNT: 0
GDM2_TX_SKIPCNT: 0
GDM2_TX_COLCNT: 0
GDM2_RX_GBCNT: 0
GDM2_RX_GPCNT: 0
GDM2_RX_OERCNT: 0
GDM2_RX_FERCNT: 0
GDM2_RX_SERCNT: 0
GDM2_RX_LERCNT: 0
GDM2_RX_CERCNT: 0
GDM2_RX_FCCNT: 0

Port 0:
	mib: Port 0 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 6861716
TxMulti    : 8
TxBroad    : 12
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 4056
Tx65Byte   : 11645
Tx128Byte  : 13210
Tx256Byte  : 249
Tx512Byte  : 169
Tx1024Byte : 6832407
TxByte     : 10238376166
RxDrop     : 0
RxFiltered : 49
RxUni      : 963037
RxMulti    : 1200795
RxBroad    : 54114
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 56679
Rx65Byte   : 117104
Rx128Byte  : 1359908
Rx256Byte  : 181766
Rx512Byte  : 198823
Rx1024Byte : 303666
RxByte     : 889985596
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 2
	link: port:0 link:up speed:100baseT full-duplex 
Port 1:
	mib: Port 1 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 948176
TxMulti    : 170
TxBroad    : 3
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 1557
Tx65Byte   : 930766
Tx128Byte  : 1302
Tx256Byte  : 528
Tx512Byte  : 75
Tx1024Byte : 14121
TxByte     : 87870052
RxDrop     : 0
RxFiltered : 0
RxUni      : 6849258
RxMulti    : 187
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 911
Rx65Byte   : 14343
Rx128Byte  : 298
Rx256Byte  : 88
Rx512Byte  : 56
Rx1024Byte : 6833749
RxByte     : 9828214886
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 1
	link: port:1 link:up speed:1000baseT full-duplex 
Port 2:
	mib: Port 2 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 0
TxMulti    : 0
TxBroad    : 0
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 0
Tx65Byte   : 0
Tx128Byte  : 0
Tx256Byte  : 0
Tx512Byte  : 0
Tx1024Byte : 0
TxByte     : 0
RxDrop     : 0
RxFiltered : 0
RxUni      : 0
RxMulti    : 0
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 0
Rx65Byte   : 0
Rx128Byte  : 0
Rx256Byte  : 0
Rx512Byte  : 0
Rx1024Byte : 0
RxByte     : 0
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 1
	link: port:2 link:down
Port 3:
	mib: Port 3 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 0
TxMulti    : 0
TxBroad    : 0
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 0
Tx65Byte   : 0
Tx128Byte  : 0
Tx256Byte  : 0
Tx512Byte  : 0
Tx1024Byte : 0
TxByte     : 0
RxDrop     : 0
RxFiltered : 0
RxUni      : 0
RxMulti    : 0
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 0
Rx65Byte   : 0
Rx128Byte  : 0
Rx256Byte  : 0
Rx512Byte  : 0
Rx1024Byte : 0
RxByte     : 0
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 1
	link: port:3 link:down
Port 4:
	mib: Port 4 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 0
TxMulti    : 0
TxBroad    : 0
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 0
Tx65Byte   : 0
Tx128Byte  : 0
Tx256Byte  : 0
Tx512Byte  : 0
Tx1024Byte : 0
TxByte     : 0
RxDrop     : 0
RxFiltered : 0
RxUni      : 0
RxMulti    : 0
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 0
Rx65Byte   : 0
Rx128Byte  : 0
Rx256Byte  : 0
Rx512Byte  : 0
Rx1024Byte : 0
RxByte     : 0
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 1
	link: port:4 link:down
Port 5:
	mib: Port 5 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 0
TxMulti    : 0
TxBroad    : 0
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 0
Tx65Byte   : 0
Tx128Byte  : 0
Tx256Byte  : 0
Tx512Byte  : 0
Tx1024Byte : 0
TxByte     : 0
RxDrop     : 0
RxFiltered : 0
RxUni      : 0
RxMulti    : 0
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 0
Rx65Byte   : 0
Rx128Byte  : 0
Rx256Byte  : 0
Rx512Byte  : 0
Rx1024Byte : 0
RxByte     : 0
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 0
	link: port:5 link:down
Port 6:
	mib: Port 6 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 7812303
TxMulti    : 1200937
TxBroad    : 54112
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 13632
Tx64Byte   : 13632
Tx65Byte   : 188400
Tx128Byte  : 1360750
Tx256Byte  : 181926
Tx512Byte  : 198875
Tx1024Byte : 7137402
TxByte     : 10755312044
RxDrop     : 0
RxFiltered : 51
RxUni      : 7809918
RxMulti    : 201
RxBroad    : 30
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 89
Rx64Byte   : 5720
Rx65Byte   : 942425
Rx128Byte  : 14535
Rx256Byte  : 778
Rx512Byte  : 249
Rx1024Byte : 6846531
RxByte     : 10357481140
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 0
	link: port:6 link:up speed:1000baseT full-duplex 
Port 7:
	mib: Port 7 MIB counters
TxDrop     : 0
TxCRC      : 0
TxUni      : 0
TxMulti    : 0
TxBroad    : 0
TxCollision: 0
TxSingleCol: 0
TxMultiCol : 0
TxDefer    : 0
TxLateCol  : 0
TxExcCol   : 0
TxPause    : 0
Tx64Byte   : 0
Tx65Byte   : 0
Tx128Byte  : 0
Tx256Byte  : 0
Tx512Byte  : 0
Tx1024Byte : 0
TxByte     : 0
RxDrop     : 0
RxFiltered : 0
RxUni      : 0
RxMulti    : 0
RxBroad    : 0
RxAlignErr : 0
RxCRC      : 0
RxUnderSize: 0
RxFragment : 0
RxOverSize : 0
RxJabber   : 0
RxPause    : 0
Rx64Byte   : 0
Rx65Byte   : 0
Rx128Byte  : 0
Rx256Byte  : 0
Rx512Byte  : 0
Rx1024Byte : 0
RxByte     : 0
RxCtrlDrop : 0
RxIngDrop  : 0
RxARLDrop  : 0

	pvid: 0
	link: port:7 link:down
VLAN 1:
	vid: 1
	ports: 1 2 3 4 6t 
VLAN 2:
	vid: 2
	ports: 0 6t 
```

WTF !? 

既然另外幾台都沒有問題，那麼應該就是這台機器的網路孔、或者網路線有問題了！

那就換換看網路線吧！

果然從原本的 CAT 5E 換成 CAT 6 之後，連線速率就變成 1000 Mb了

但是CAT 5E 應該要能支援到1000Mb 才對啊！

所以就是這條 CAT 5E 要不就是偷工減料，要不就是年紀到了，衰退了？？

以後還是不要用  CAT 5E 的線了...

這邊太多的古董，總是藏著一些奇奇怪怪的臭蟲 ....

同場加映一下 wireguard 連線的速率

大概都能跑到200 Mb 左右

比起原本strongswan 打的 IPSEC 只有 30 Mb 左右 ，那是進步太多太多了！

strongswan 的設定又囉唆，該是讓他退場的時候了！

![](https://i.imgur.com/QwQLH2V.png)






=== delete below content when finish the post ===
youtbe: {{< youtube w7Ft2ymGmfc >}} 
IG photo: {{< instagram BWNjjyYFxVx >}}

