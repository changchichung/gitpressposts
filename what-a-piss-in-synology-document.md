---
title: "[碎念] Synology 群暉的文件不知道在工三小 / what a piss in synology document"
date: 2020-01-03T11:45:56+08:00
draft: false
noSummary: false
categories: ['雜念']
image: https://h.cowbay.org/images/post-default-16.jpg
tags: ['synology']
author: "Eric Chang"
keywords:
  - synology
---

2020/01/02  2020 上工的第一天，公司碩果僅存的唯一一台 Synology DS415+ 也終於掛了

開機沒多久就連不上，反覆幾次之後，出現了開機時所有燈號都狂閃的狀況

終於宣告不治

問題很明顯的就是Intel C2000 系列 CPU 的瑕疵

<!--more-->

總之，機器老早就過保了，上面放的是 proxmox 的 vm 檔案

在NAS掛點之後，就從備份檔把這些VM還原回來了

想說網路上很多文章說只要焊一個電阻上去就可以修復

就把機器和硬碟先放著，等有空再去買電阻回來玩玩看

結果user今天早上就在靠腰，說上面有一台開發用的VM上面的歷史紀錄很重要

幹，很重要是不會自己備份逆？

又不跟我說很重要，要備份，然後自己也不做備份

然後現在VM 不見了，再來靠腰？？

幹！真的不要以為資訊公司的員工就比較有sense ，屁！

不過呢，人微言輕，還是只好鼻子摸摸，想辦法救出來

然後就找到了群暉的這篇文章

https://www.synology.com/zh-tw/knowledgebase/DSM/tutorial/Storage/How_can_I_recover_data_from_my_DiskStation_using_a_PC

```
如何使用電腦復原存放在 Synology NAS 上的資料？

若您的 Synology NAS 故障，可以輕鬆透過電腦與 Ubuntu Live CD 復原資料。請確認 Synology NAS 硬碟上運行的檔案系統是 EXT4 或 Btrfs，並依照下列步驟來復原資料。此處將以 Ubuntu 18.04 版本作為範例：

1.準備一台電腦，該電腦必須具備足夠的硬碟插槽以安裝從 Synology NAS 取出的硬碟。
2.將硬碟從 Synology NAS 取出，並安裝到電腦。若使用 RAID 或 SHR 配置，您必須將所有硬碟 (Hot Spare 硬碟除外) 同時安裝到電腦。
3.按照此教學 Create a bootable USB stick on Windows 來建立 Ubuntu 環境。
4.前往左下角的顯示應用程式選單。
5.在搜尋欄位輸入 Terminal 並選擇終端機。
6.若 Synology NAS 上的磁碟配置為 RAID 或 SHR，請依照步驟 7 到 10 操作；若您想復原的檔案位於僅使用一顆硬碟的基本儲存類型機種，請跳至步驟 10。
7.輸入以下指令 (sudo 會將執行權限轉換為 root )。

    Ubuntu@ubuntu:~$ sudo -i

8.輸入以下指令來安裝 mdadm 和 lvm2 (皆為 RAID 管理工具)。若沒有安裝 lvm2，vgchange 將無法運作。

    root@ubuntu:~$ apt-get update
    root@ubuntu:~$ apt-get install -y mdadm lvm2

9.輸入以下指令來掛載所有從 Synology NAS 取出的硬碟，結果可能會因 Synology NAS 上的儲存集區配置而有所不同。

    root@ubuntu:~$ mdadm -Asf && vgchange -ay

10.輸入以下指令來將所有硬碟掛載為唯讀以存取資料。在 ${device_path} 輸入裝置路徑，${mount_point} 輸入掛載點，您的資料將會被置於掛載點的路徑。

    $ mount ${device_path} ${mount_point} -o ro
```

好， 1-9 都沒什麼問題，但是有人可以幫忙翻譯翻譯 10 是在工三小？

當然，我能理解因為每一臺NAS的環境不同，所以會有一些不同的變數

但是假如你是一個單純的user ，只是想要救資料，好不容易找了臺電腦

把硬碟都接上去，用ubuntu liveCD 開機，乖乖做了1-9的步驟

接著一定會傻眼， 什麼是 ${device_path} ?? 什麼是 ${mount_point} ???

寫文件的人你就不能配合個圖片，去說明應該要怎麼辨別 device_path ? mount_point 又是什麼？

這很簡單呀！

做完 9 的指令，其實就會回復你 NAS 分割區的名稱

好像叫什麼 vg1 的 <---這個就是變數，可能每一臺都不同，但是你起碼做個範例給人家看啊！

然後會在 /dev/vg1 底下看到當初建立的磁區 (我的叫 volume_1)

至於 mount_point 就是看你要掛載到系統的哪個目錄底下

所以我就要執行
```
mount /dev/vg1/volume_1 /mnt
```

這樣就可以把NAS上的分割給掛進liveCD ，就可以進行資料複製了！

連一份文件都做不好，真的是服了這些據說很高薪的「工程師」..




