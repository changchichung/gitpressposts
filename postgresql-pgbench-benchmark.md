---
title: "[筆記] postgresql 效能測試 / postgresql benchmakr using pgbench"
date: 2020-01-07T11:18:59+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-17.jpg
tags: ['postgresql','pgbench']
author: "Eric Chang"
keywords:
  - postgresql
  - pgbench
---

昨天老闆在slack 上面問說現在的幾台 DB Server 有沒有跑過 pgbench

分數大概如何，想要跟他的筆電做個比較

之前有跑過幾次，這次就順便測試一下不同的硬體配置、以及不同的軟體版本

對於pgbench 跑分會有多大的影響

<!--more-->

OS: ubuntu 18.04.3 x64
postgresql 版本： 10 / 11 / 12
硬碟分成兩種，一個是透過 NFS 10G 網路存取的storage，一個是本機三顆硬碟組成的 zfs raidz

大概步驟就是安裝postgresql & tools ，然後initialize pgbench table 最後就跑pgbench 測試

### install tools for postgresql

sudo apt install postgresql-contrib

### su to postgres and initialize pgbench database

sudo su - postgres
createdb pgbench
pgbench -i -U postgres -s 10 pgbench

### running the test

pgbench -t 100 -c 100 -S -U postgres pgbench


得出來的結果如下

|  | 2 cores / 16G | 4 cores / 16G |
| --- | --- | --- | PGTUNE | NO PGTUNE | PGTUNE | NO PGTUNE |
| PSQL Version | 10G Storage | Local Raidz | 10G Storage | Local Raidz | 10G Storage | Local Raidz | 10G Storage | Local Raidz |
| 10 | 9014.144993 | 9395.847239 | 9508.819462 | 10192.27069 | 13280.99918 | 13819.12767 | 15257.69002 | 15397.53475 |
| 11 | 9418.477212 | 9333.790266 | 9070.990565 | 9071.182748 | 15455.80444 | 16079.6638 | 15710.24677 | 14274.59939 |
| 12 | 8630.21746 | 8872.475173 | 9072.034237 | 9217.547833 | 16116.7502 | 12380.71452 | 17409.10363 | 14520.79393 |

Update: 喵的 Markdown 的表格不支援 colspan ，只好改用圖片方式呈現

!['postgresql pgbench banchmark reults'](https://i.imgur.com/vQFfj6Y.png)


另外補上一個 2 cores / 2G RAM 的結果
### postgresql 10 , 2G RAM , HDD on 10G Storage

```
postgres@ubuntu:~$ pgbench -t 100 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 100
number of transactions actually processed: 10000/10000
latency average = 11.583 ms
tps = 8633.209610 (including connections establishing)
tps = 8651.036900 (excluding connections establishing)
```

有幾個地方值得注意

* 記憶體 2G->16G 效能的增加並沒有很明顯 tps 從 8633 略為上升到 9014
  
  * 這個倒是讓我滿意外的，一直以來都認為postgresql 非常的需要記憶體，但是實際跑測試卻不是這樣

* pgtune 的影響不大，甚至可以說是會降低效能

  * pgtune 是一個網頁服務，可以協助做出「理論上」建議使用的postgresql config 
  https://pgtune.leopard.in.ua/#/
  * 從結果可以看出，使用pgtune 做出來的config ，跟完全使用預設值的config 相比，pgtune的效能大部分都略低於預設值
  * 這也讓我很好奇，或許要花更多時間去研究postgresql 的config，但是，幹！我不是 DBA 啊！
* CPU 核心數很明顯地影響pgbench

  * 從表格中可以看到，當CPU Cores 增加，pgbench的效能也明顯增加
  * 而我甚至還沒有指定用多核心去執行測試，如果要用多核心去測試，要把測試指令改成
  ```
  pgbench -j 4 -t 100 -c 100 -S -U postgres pgbench
  
  ```

* 10G Storage和 3顆 2T SATA硬碟組成的 raidz 效能差不多
  
  * 如果本機改用 SSD RAID 甚至是 NVME SSD RAID ，效能應該會提高更多
  * 10G的部份最多大概就是略低於 1000MB 左右
  * 如果換成 SSD ，效能應該是還會提昇，但是有限，畢竟10Gb的頻寬限制就在那邊(理論值1250MB左右)


