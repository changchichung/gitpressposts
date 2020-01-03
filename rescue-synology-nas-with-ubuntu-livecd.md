---
title: "[筆記] 用ubuntu livecd 救援群暉 synology NAS內的資料 / rescue synology nas with ubuntu livecd"
date: 2020-01-03T15:43:45+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-11.jpg
tags: ['synology','nas']
author: "Eric Chang"
keywords:
  - synology
  - nas
---

2020/01/02 , 2020年上工的第一天，群暉的 DS415+ NAS 掛了！

因為群暉的文件在最關鍵的一步寫得亂七八糟！

所以在這邊紀錄一下我操作的步驟！

<!--more-->

#### 建立可開機的ubuntu 隨身碟
建立 bootable ubuntu flash 的步驟，請參考底下網頁介紹，這邊就不多說了

https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-ubuntu#0

#### 把NAS上的硬碟接上PC

還好這次的NAS只有四顆，如果有八顆，我去哪裡生可以接八顆硬碟的主機...

#### 用隨身碟開機進入ubuntu Live 環境

懶人沒截圖

#### 安裝必要套件

進入 ubuntu Live 之後，按 ctal + alt + t

開啟 terminal ，然後先安裝 mdadm & lvm2 

```
ubuntu@ubuntu:~$ sudo apt install mdadm lvm2
Reading package lists... Done
Building dependency tree       
Reading state information... Done
Suggested packages:
  thin-provisioning-tools default-mta | mail-transport-agent dracut-core
The following NEW packages will be installed:
  mdadm
The following packages will be upgraded:
  lvm2
1 upgraded, 1 newly installed, 0 to remove and 780 not upgraded.
Need to get 1,346 kB of archives.
After this operation, 1,237 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 lvm2 amd64 2.02.176-4.1ubuntu3.18.04.2 [930 kB]
Get:2 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 mdadm amd64 4.1~rc1-3~ubuntu18.04.2 [416 kB]
Fetched 1,346 kB in 3s (501 kB/s)
....
...
...
以下省略
```

#### scan raid and lvm

接下來先換成 root 操作
```
ubuntu@ubuntu:~$ sudo su -
```

然後掃描 raid & LVM
```
root@ubuntu:~# mdadm -Asf && vgchange -ay
mdadm: /dev/md/2 has been started with 4 drives.
  2 logical volume(s) in volume group "vg1" now active
```

COOL！ 原本的VG出現了！

```
root@ubuntu:~# vgdisplay
  --- Volume group ---
  VG Name               vg1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               5.44 TiB
  PE Size               4.00 MiB
  Total PE              1427264
  Alloc PE / Size       1427264 / 5.44 TiB
  Free  PE / Size       0 / 0   
  VG UUID               O1c8Uw-JmKy-EiKt-92OB-3K3y-roMi-9NUZ6H
```
 
也可以看到 RAID 資訊了！
```
root@ubuntu:~# mdadm -D /dev/md2
/dev/md2:
           Version : 1.2
     Creation Time : Thu Oct 13 07:26:12 2016
        Raid Level : raid5
        Array Size : 5846077632 (5575.25 GiB 5986.38 GB)
     Used Dev Size : 1948692544 (1858.42 GiB 1995.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan  2 01:48:34 2020
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 64K

Consistency Policy : resync

              Name : video:2
              UUID : 18f6706d:91eaaec9:5b0ba8da:e32481e3
            Events : 96

    Number   Major   Minor   RaidDevice State
       0       8       51        0      active sync   /dev/sdd3
       1       8       35        1      active sync   /dev/sdc3
       2       8       19        2      active sync   /dev/sdb3
       3       8        3        3      active sync   /dev/sda3
```	   

然後就會發生我之前寫的這篇的狀況

https://h.cowbay.org/post/what-a-piss-in-synology-document/

問題發生了，總是要想辦法解決

#### scan lv

```
root@ubuntu:~# lvscan
  ACTIVE            '/dev/vg1/syno_vg_reserved_area' [12.00 MiB] inherit
  ACTIVE            '/dev/vg1/volume_1' [5.44 TiB] inherit
```  

OK ，在 vg1 底下有兩個 volume ，看大小來判斷，第二個是我們要的

用底下的指令就可以掛載了

```
mount /dev/vg1/volume_1 /mnt
```

請依照自己的環境，把第一個路徑改掉，如果要掛載到別的目錄，那也把第二個 /mnt 改掉

```
root@ubuntu:/dev# mount /dev/vg1/volume_1 /mnt
root@ubuntu:/dev# cd /mnt
root@ubuntu:/mnt# ls
@appstore     @database  @EP_trash   @MailScanner  @S2S
aquota.group  @download  @iSCSITrg   music         synoquota.db
aquota.user   @eaDir     lost+found  nfsforprox    @tmp
@clamav       @EP        @maillog    photo         video
```

OK，可以看到原本NAS 下的目錄了，接下來就可以進行檔案複製了！




