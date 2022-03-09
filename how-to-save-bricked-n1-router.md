---
title: "[筆記] Phicomm 斐訊 N1 救磚的方法/ How to Save Bricked N1 Router"
date: 2022-03-09T16:29:17+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-14.jpg
tags: ['n1','router']
author: "Eric Chang"
keywords:
  - N1
  - router
---

不久前買了一台對岸斐訊出的 N1 路由器，這台的規格很強，又有大神破解了 boot loader

所以可以被拿來安裝 openwrt/Armbian 之類的系統做其他的應用

因為openwrt 玩很多了，所以這次想說來試試看 Armbian

一開始只弄出了based on ubuntu bionic 的版本，因為覺得有點舊了，所以一直想要換成 ubuntu focal

就在某次亂搞之後，N1 他變磚了...開機完全沒有畫面，只好開始研究怎麼救磚了

<!--more-->

### N1 救磚的方法

筆記日期： 2022-03-04 17:15

這個是已經確認 N1 變磚了，開機沒有boot loader 的狀況

這方法不需要拆機，需要用到的檔案和線材如下

1. 雙公頭 USB $40-50 , shopee 買的
2. 02_Amlogic_USB_Burning_Tool.tgz [點我下載](https://nextcloud.slat.org/index.php/s/PC72sNQDwmk8tZ8)
3. 03_aml_upgrade_package.tgz [點我下載](https://nextcloud.slat.org/index.php/s/KTPWgrSJweZpn2E)
4. 04_balenaEtcher-Setup-1.7.7.exe [點我下載](https://nextcloud.slat.org/index.php/s/z8GjfErnYnZZrQQ)
6. 05_platform-tools_r33.0.0-windows.zip [點我下載](https://nextcloud.slat.org/index.php/s/pRcGTYzFeLGdQMr)
7. 06_T1_1.3T47_mod_by_webpad_v3_20180419_2.tgz [點我下載](https://nextcloud.slat.org/index.php/s/JGzqYwFzm2tG5Yx)
8. 一台Windows 筆電/桌機

簡單描述一下我還記得的步驟，就不上圖了，反正重要的是這些檔案，操作過程其實容易的

先把 02/03/06 的檔案傳到筆電上，該解壓縮的，該安裝的都做一做
(在這邊至少要確定驅動程式有安裝成功，叫什麼WorldCup 有的沒的)

02 的檔案解壓縮完會有一個license 的目錄，這個目錄要複製到 USB Burning Tool 的安裝目錄底下
一般來說應該就是

```
c:\program files(x86)\Amlogic\USB_BURNING_TOOL
```

確認複製好、解壓縮也完成了，這時候拿出 N1 、雙公頭USB、電源線並且執行 USB Burning Tool

進入USB Burning Tool 畫面，點一下 File --> Import image

選擇06 解壓縮的 06_T1_1.3T47_mod_by_webpad_v3_20180419_2.img

因為我的N1 是連 Bootloader 都掛了，所以右邊有四個選項要勾選
1. Erase flash 
2. Erase bootloader
3. Overwrite key
4. Secure Boot Key (跟圖片上的不會一樣)

還是補個圖好了，借別人的來用

![](https://i.imgur.com/kUibBJY.png)


確認好之後，還先不要按開始，準備好 N1，電源接上插座，但先不要接到N1，USB 一頭接在電腦上，另一頭也是一樣先不要接到 N1

接下來點一下USB Burning tool 的開始，然後快速的先插入電源接頭到 N1 ，然後接著插入 USB 接頭

正常的話，這邊就會聽到聲音，然後 USB Burning Tool 就會開始燒錄，如果沒有看到開始燒錄的話，就多試幾次看看，

我一開始就弄錯 USB 和電源的插入順序，一直沒成功

在燒錄到21% 的時候，會跳出錯誤訊息，不用擔心，意料之中，

這時候按下停止，然後一樣到 File --> import image，這次選擇 03 解壓縮後的檔案aml_upgrade_package，

右邊的選項也不用動，密鑰的部分會自動消失，然後保留上面兩個Erase flash/bootloader ，再點一次開始按鈕

基本上這次就可以順利成功燒錄了，如果只是要救磚，到這邊就已經 OK 

這時候的機子韌體版本是 V2.19 ，如果想要可以繼續刷其他的韌體

### 刷 Armbian 22.10 focal kernel 5.9

這部分已經成功了，只是還沒整理筆記(其實也沒很複雜啦)

等到有心情想寫的時候再來補
