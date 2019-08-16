---
title: "[碎念] mdadm 超級慢的rebuild 速度 Incredibly Slow mdadm Rebuild"
date: 2018-12-12T11:10:22+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-1.jpg"
categories: ['碎念']
tags: ['mdadm']
author: "Eric Chang"
---

最近在做一台老機器的P2V

偏偏user說不能關機，所以我用dd + ssh 做線上移轉

這部份有空再來寫

只是因為原來的設定有用mdadm 做raid1 

這部份導致移轉過去proxmox 後，會出現raid degrade 導致無法正常開機

<!--more-->

我的想法是既然開機會出現raid degrade

那我就加第二顆硬碟給它，讓它做 rebuild

OK，原則上這樣做沒有問題

問題是它X的，這個mdadm rebuild 的速度也未免太慢了吧！

```
Every 2.0s: cat /proc/mdstat                                                                                                        Wed Dec 12 11:01:36 2018

Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 sda1[0] sdb1[2]
      975296 blocks super 1.2 [2/2] [UU]

md0 : active raid1 sda5[0] sdb5[2]
      487276352 blocks super 1.2 [2/1] [U_]
      [>....................]  recovery =  0.5% (2468032/487276352) finish=2079.7min speed=3884K/sec

unused devices: <none>
```

這個是一開始跑的速度

然後這個是跑了大概10分鐘之後的速度(有稍稍提昇一點點)

```
Every 2.0s: cat /proc/mdstat                                                                                                        Wed Dec 12 11:17:31 2018

Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 sda1[0] sdb1[2]
      975296 blocks super 1.2 [2/2] [UU]

md0 : active raid1 sda5[0] sdb5[2]
      487276352 blocks super 1.2 [2/1] [U_]
      [>....................]  recovery =  1.4% (6978688/487276352) finish=1915.9min speed=4177K/sec

unused devices: <none>
```

等等看會不會速度再快一點..

一個小時了，還是一樣慢..
```
Every 2.0s: cat /proc/mdstat                                                                                                        Wed Dec 12 12:20:16 2018

Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 sda1[0] sdb1[2]
      975296 blocks super 1.2 [2/2] [UU]

md0 : active raid1 sda5[0] sdb5[2]
      487276352 blocks super 1.2 [2/1] [U_]
      [=>...................]  recovery =  5.2% (25526016/487276352) finish=858.6min speed=8962K/sec

unused devices: <none>

```

突然又加速了，希望能維持下去啊！....

```
Every 2.0s: cat /proc/mdstat                                                                                                        Wed Dec 12 13:33:42 2018

Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md1 : active raid1 sda1[0] sdb1[2]
      975296 blocks super 1.2 [2/2] [UU]

md0 : active raid1 sda5[0] sdb5[2]
      487276352 blocks super 1.2 [2/1] [U_]
      [========>............]  recovery = 43.6% (212611904/487276352) finish=47.6min speed=96104K/sec

unused devices: <none>

```

