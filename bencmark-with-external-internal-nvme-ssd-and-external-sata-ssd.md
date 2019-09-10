---
title: "[筆記] 測試 USB 3.1 Gen2 NVME SSD 外接盒 & 內建pci-e ssd & 外接SATA SSD / Bencmark With External Internal Nvme Ssd and External Sata Ssd"
date: 2019-09-10T14:37:09+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['postgresql','benchmark','nvme']
author: "Eric Chang"
keywords:
  - 'USB 3.1 Gen2 NVME SSD'
  - 'NVME SSD'
  - 'benchmark'
---

前幾天在淘寶上買了個 SSK 的USB 3.1 Gen2 (type-c) NVME SSD 外接盒
手邊也剛好有一條多的intel 600p nvme ssd 就順手來做個比較
目標是看看有沒有可能直接用外接的SSD來跑postgresql 

<!--more-->
把600p 裝進去外接盒之後，就先來看一些簡單的資訊
不過沒想到用了幾個指令，都沒辦法辨別出正確的型號

#### fdisk
沒有看到廠牌、型號
```
2019-09-10 13:20:55 [minion@hqdc075 ~]$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): p
Disk /dev/sdb: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x59511d8e

Device     Boot Start       End   Sectors   Size Id Type
/dev/sdb1        2048 500118191 500116144 238.5G 83 Linux

Command (m for help): q
```

#### inxi 
內接的intel ssd 有看到，不過外接的這顆SSD沒有model 可是可以看到序號
我在想這個序號應該是外接盒的序號，而不是SSD的？

```
2019-09-10 13:21:51 [minion@hqdc075 ~]$ sudo inxi -Dxx
Drives:    HDD Total Size: 640.2GB (11.3% used)
           ID-1: /dev/nvme0n1 model: INTEL_SSDPEKKF256G7L size: 256.1GB serial: BTPYXXXXX firmware: 123P
           ID-2: USB /dev/sda model: SD/MMC size: 128.1GB serial: 201205XXXXXX-0:0 temp: 0C
           ID-3: USB /dev/sdb model: N/A size: 256.1GB serial: DF564XXXXXX:0 temp: 0C
2019-09-10 13:21:55 [minion@hqdc075 ~]$
```

#### dmesg
一樣，也是認不出 SSD ，但是有抓到外接盒

```
[16622.930915] hub 4-0:1.0: USB hub found
[16622.930926] hub 4-0:1.0: 2 ports detected
[16623.372533] usb 4-1: new SuperSpeedPlus Gen 2 USB device number 2 using xhci_hcd
[16623.393844] usb 4-1: New USB device found, idVendor=152d, idProduct=0562, bcdDevice= 2.04
[16623.393849] usb 4-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[16623.393853] usb 4-1: Product: SSK Storage
[16623.393856] usb 4-1: Manufacturer: SSK
[16623.393858] usb 4-1: SerialNumber: DF56419883B20
```

#### 直接測試吧

不看型號了，直接測試吧！先切好分割、格式化，然後掛載到 /mnt，再用 dd 測試寫入
結果如下

```
2019-09-10 13:24:03 [minion@hqdc075 ~]$ for i in {4,8,16,32,64,128,256,512,1024,2048,4096,8192,16384};do sudo dd if=/dev/zero of=/mnt/"$i"k bs="$i"k count=1k;done
1024+0 records in
1024+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.013719 s, 306 MB/s
1024+0 records in
1024+0 records out
8388608 bytes (8.4 MB, 8.0 MiB) copied, 0.0123268 s, 681 MB/s
1024+0 records in
1024+0 records out
16777216 bytes (17 MB, 16 MiB) copied, 0.0196891 s, 852 MB/s
1024+0 records in
1024+0 records out
33554432 bytes (34 MB, 32 MiB) copied, 0.0195221 s, 1.7 GB/s
1024+0 records in
1024+0 records out
67108864 bytes (67 MB, 64 MiB) copied, 0.0337692 s, 2.0 GB/s
1024+0 records in
1024+0 records out
134217728 bytes (134 MB, 128 MiB) copied, 0.0644939 s, 2.1 GB/s
1024+0 records in
1024+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 0.131989 s, 2.0 GB/s
1024+0 records in
1024+0 records out
536870912 bytes (537 MB, 512 MiB) copied, 0.257682 s, 2.1 GB/s
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB, 1.0 GiB) copied, 0.529154 s, 2.0 GB/s
1024+0 records in
1024+0 records out
2147483648 bytes (2.1 GB, 2.0 GiB) copied, 3.48498 s, 616 MB/s
1024+0 records in
1024+0 records out
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 7.36899 s, 583 MB/s
1024+0 records in
1024+0 records out
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 73.1975 s, 117 MB/s
1024+0 records in
1024+0 records out
17179869184 bytes (17 GB, 16 GiB) copied, 153.142 s, 112 MB/s
2019-09-10 13:28:58 [minion@hqdc075 ~]$
```

可以發現，**當寫入8G檔案的時候，速度開始急遽下降**
翻了一下google 看到這張表格

https://www.anandtech.com/show/10850/the-intel-ssd-600p-512gb-review

intel 600p 256G 的SLC Cache 只有8.5GB
合理解釋了為什麼當寫入8G的檔案時，速度會掉那麼慘，跟SATA硬碟差不多了

#### 用iobench 測試看看

得到一樣的結論，8G左右，就會把cache塞暴，然後掉速
```
2019-09-10 13:44:59 [changch@hqdc075 iobench]$ sudo ./iobench.sh --sync --dir /mnt --megabytes 8192
Target directory: /mnt
Testfile size: 8192 x 1 Megabyte

1. Write benchmark without cache
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 36.8695 s, 233 MB/s

2. Write benchmark with cache
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 64.1573 s, 134 MB/s

3. Read benchmark with dropped cache
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 9.21821 s, 932 MB/s

4. Read benchmark without cache drop

Start 1 of 5...
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 1.02253 s, 8.4 GB/s

Start 2 of 5...
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 0.832002 s, 10.3 GB/s

Start 3 of 5...
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 0.81194 s, 10.6 GB/s

Start 4 of 5...
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 0.821035 s, 10.5 GB/s

Start 5 of 5...
8589934592 bytes (8.6 GB, 8.0 GiB) copied, 0.808803 s, 10.6 GB/s

Done.
2019-09-10 13:47:40 [changch@hqdc075 iobench]$
```

那如果不用8G的檔案大小來測試呢？看起來是比較正常一點
```
2019-09-10 13:47:40 [changch@hqdc075 iobench]$ sudo ./iobench.sh --sync --dir /mnt --megabytes 4096
Target directory: /mnt
Testfile size: 4096 x 1 Megabyte

1. Write benchmark without cache
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 7.55296 s, 569 MB/s

2. Write benchmark with cache
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 20.6237 s, 208 MB/s

3. Read benchmark with dropped cache
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 4.63622 s, 926 MB/s

4. Read benchmark without cache drop

Start 1 of 5...
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 0.518597 s, 8.3 GB/s

Start 2 of 5...
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 0.436877 s, 9.8 GB/s

Start 3 of 5...
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 0.437854 s, 9.8 GB/s

Start 4 of 5...
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 0.420224 s, 10.2 GB/s

Start 5 of 5...
4294967296 bytes (4.3 GB, 4.0 GiB) copied, 0.431023 s, 10.0 GB/s

Done.
2019-09-10 13:50:09 [changch@hqdc075 iobench]$
```

#### 跑看看pgbench ??

試試看這次的目標，用 external nvme ssd 跑資料庫，會不會跟放在本機的PCI-E SSD 有所差距？

#### data_directory in internal pcie-ssd

**initialize pgbench database**
```
2019-09-10 13:52:35 [minion@hqdc075 ~]$ sudo su - postgres
postgres@hqdc075:~$ createdb pgbench
postgres@hqdc075:~$ pgbench -i -U postgres -s 10 pgbench
NOTICE:  table "pgbench_history" does not exist, skipping
NOTICE:  table "pgbench_tellers" does not exist, skipping
NOTICE:  table "pgbench_accounts" does not exist, skipping
NOTICE:  table "pgbench_branches" does not exist, skipping
creating tables...
100000 of 1000000 tuples (10%) done (elapsed 0.08 s, remaining 0.70 s)
200000 of 1000000 tuples (20%) done (elapsed 0.19 s, remaining 0.75 s)
300000 of 1000000 tuples (30%) done (elapsed 0.36 s, remaining 0.83 s)
400000 of 1000000 tuples (40%) done (elapsed 0.49 s, remaining 0.73 s)
500000 of 1000000 tuples (50%) done (elapsed 0.58 s, remaining 0.58 s)
600000 of 1000000 tuples (60%) done (elapsed 0.75 s, remaining 0.50 s)
700000 of 1000000 tuples (70%) done (elapsed 0.89 s, remaining 0.38 s)
800000 of 1000000 tuples (80%) done (elapsed 0.99 s, remaining 0.25 s)
900000 of 1000000 tuples (90%) done (elapsed 1.11 s, remaining 0.12 s)
1000000 of 1000000 tuples (100%) done (elapsed 1.27 s, remaining 0.00 s)
vacuum...
set primary keys...
done.
```

**run pgbench**
```
postgres@hqdc075:~$ pgbench -t 10 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 1000/1000
latency average = 32.118 ms
tps = 3113.559459 (including connections establishing)
tps = 3135.056341 (excluding connections establishing)
postgres@hqdc075:~$ pgbench -t 1000 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 1000
number of transactions actually processed: 100000/100000
latency average = 3.753 ms
tps = 26643.223990 (including connections establishing)
tps = 26659.061459 (excluding connections establishing)
```


#### data_directory in external NVME SSD

**initialize pgbench database**
```
postgres@hqdc075:~$ pgbench -i -U postgres -s 10 pgbench
creating tables...
100000 of 1000000 tuples (10%) done (elapsed 0.08 s, remaining 0.70 s)
200000 of 1000000 tuples (20%) done (elapsed 0.19 s, remaining 0.76 s)
300000 of 1000000 tuples (30%) done (elapsed 0.35 s, remaining 0.81 s)
400000 of 1000000 tuples (40%) done (elapsed 0.49 s, remaining 0.74 s)
500000 of 1000000 tuples (50%) done (elapsed 0.60 s, remaining 0.60 s)
600000 of 1000000 tuples (60%) done (elapsed 0.76 s, remaining 0.51 s)
700000 of 1000000 tuples (70%) done (elapsed 0.89 s, remaining 0.38 s)
800000 of 1000000 tuples (80%) done (elapsed 1.01 s, remaining 0.25 s)
900000 of 1000000 tuples (90%) done (elapsed 1.15 s, remaining 0.13 s)
1000000 of 1000000 tuples (100%) done (elapsed 1.32 s, remaining 0.00 s)
vacuum...
set primary keys...
done.
```

**run pgbench**
可以看到兩邊的結果其實是差不多的，作為資料庫備份是一定沒有問題
至於能不能直接作為資料庫空間使用？我想也許可以嘗試看看..

```
postgres@hqdc075:~$ pgbench -t 10 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 1000/1000
latency average = 32.998 ms
tps = 3030.531670 (including connections establishing)
tps = 3051.744292 (excluding connections establishing)
postgres@hqdc075:~$ pgbench -t 1000 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 1000
number of transactions actually processed: 100000/100000
latency average = 3.758 ms
tps = 26610.771423 (including connections establishing)
tps = 26626.871871 (excluding connections establishing)
postgres@hqdc075:~$
```

update:

剛剛看了一下，發現這樣只有測試到 read (select only)
或許要找找看其他的測試方法

***

接下來直接用 restore 測試

```
#### restore 5G database with external nvme ssd
postgres@hqdc075:~$ createdb demo
postgres@hqdc075:~$ time psql demo < /tmp/demo.sql
SET
...
ALTER TABLE

real	4m1.184s
user	0m2.894s
sys	0m0.504s
postgres@hqdc075:~$ 
```

#### restore 5G database with internal pci-e nvme ssd
```
postgres@hqdc075:~$ dropdb demo
postgres@hqdc075:~$ createdb demo
postgres@hqdc075:~$ time psql demo < /tmp/demo.sql
SET
...
ALTER TABLE

real	4m1.636s
user	0m2.909s
sys	0m0.612s
postgres@hqdc075:~$ 
```

看起來是沒有什麼區別，那如果是外接的 SATA SSD呢？

*** 

dmesg 看得到型號耶..

```
[23995.478928] usb 2-4: new SuperSpeed Gen 1 USB device number 3 using xhci_hcd
[23995.506134] usb 2-4: New USB device found, idVendor=2109, idProduct=0715, bcdDevice= 3.36
[23995.506141] usb 2-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[23995.506145] usb 2-4: Product: 30848
[23995.506149] usb 2-4: Manufacturer: Ugreen
[23995.506153] usb 2-4: SerialNumber: 00000012526D
[23995.512813] scsi host2: uas
[23995.530161] scsi 2:0:0:0: Direct-Access     SanDisk  SDSSDXP240G      R131 PQ: 0 ANSI: 6
[23995.531948] sd 2:0:0:0: Attached scsi generic sg2 type 0
[23995.533820] sd 2:0:0:0: [sdc] 468862128 512-byte logical blocks: (240 GB/224 GiB)
[23995.533986] sd 2:0:0:0: [sdc] Write Protect is off
[23995.533992] sd 2:0:0:0: [sdc] Mode Sense: 2f 00 00 00
[23995.534402] sd 2:0:0:0: [sdc] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[23995.534844] sd 2:0:0:0: [sdc] Optimal transfer size 33553920 bytes
[23995.540806]  sdc:
[23995.544089] sd 2:0:0:0: [sdc] Attached SCSI disk
2019-09-10 14:24:54 [minion@hqdc075 ~]$ 
```

**restore 5G database with external sata SSD**

很意外的，速度居然還是差不多？？

```
postgres@hqdc075:~$ createdb demo
postgres@hqdc075:~$ time psql demo < /tmp/demo.sql
SET
...
ALTER TABLE

real	4m9.950s
user	0m2.752s
sys	0m0.640s
postgres@hqdc075:~$
```
**run pgebnech**

這個差很多了，tps 從前面的 26000 掉到剩下 2800 ，差十倍左右！

```
postgres@hqdc075:~$ pgbench -t 10 -c 100 -S -U postgres pgbench
starting vacuum...end.
transaction type: <builtin: select only>
scaling factor: 10
query mode: simple
number of clients: 100
number of threads: 1
number of transactions per client: 10
number of transactions actually processed: 1000/1000
latency average = 35.267 ms
tps = 2835.526770 (including connections establishing)
tps = 2855.642604 (excluding connections establishing)
postgres@hqdc075:~$
```

疑問：

1. 什麼原因會讓外接/內建 nvme ssd 和外接SATA SSD 在 restore db時，花了差不多時間，但是 tps 差異那麼大呢？
2. 這樣測試，似乎沒有真正測出外接nvme ssd 跑資料庫時候的效能？

