---
title: "[筆記] 用pbackrest 備份還原 postgresql / Backup Restore Postgresql With Pgbackrest"
date: 2019-09-05T11:42:28+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['postgresql']
author: "Eric Chang"
keywords:
  - postgresql
  - pgbackrest
---

這兩天在測試pgbackrest ，簡單筆記一下測試狀況

<!--more-->

#### install 
在ubuntu 18.04 安裝pgbackrest 很簡單，內建在apt裡面，所以可以直接用
```
sudo apt install pgbackrest
```
進行安裝

#### config

pgbackrest 的設定檔在 /etc/pgbackrest/pgbackrest.conf
如果是用apt 安裝，預設會建立一個 /etc/pgbackrest.conf
但是這個路徑是錯誤的
執行 pgbackrest的時候，預設還是會去讀取 /etc/pgbackrest/pgbackrest.conf
要特別注意，要不就是每次都指定config路徑，要不就是把那個錯誤的設定檔幹掉

#### config 內容
內容其實很簡單
```
postgres@hqdc039:~$ cat /etc/pgbackrest/pgbackrest.conf 
[demo]
pg1-path=/database/11/main

[global]
repo1-cipher-pass=zWaf6XtpjIVZC5444yXB+cgFDFl7MxGlgkZSaoPvTGirhPygu4jOKOXf9LO4vjfO
repo1-cipher-type=aes-256-cbc
repo1-path=/var/lib/pgbackrest
repo1-retention-full=2

[global:archive-push]
compress-level=3
process-max=4
```

**[demo]** 用來指定這個 "stanza" 的名稱
* pg1-path 是資料庫存放的路徑

**[global]** 中的兩個cipher 用途我不清楚，不設定也沒關係

* "repo1-path" 則是用來存放備份的路徑
* "repo1-retention-full" 定義要保留幾次 full backup

**[global:archive-push]** 似乎是用來定義在備份/還原時的選項
* process-max 指定要用多少process (平行處理)

#### 簡單流程

* config (pgbackrest & postgresql )
* 建立 stanza
* 建立備份
* (還原，如果有需要的話)

#### 其他部份
* 清除 stanza (刪除備份)
* 踩到的地雷


***

#### 設定 pgbackrest & postgresql 

pgbackrest 基本上設定很簡單，內容就跟上面的一樣就可以跑了。
當然要新增其他進階的，就要自己研究了，內容有夠長
https://pgbackrest.org/configuration.html

在postgresql 中，要新增一些設定，主要是 archive_command
```
postgres@hqdc039:~$ grep -A 100 "for pgbackrest" /etc/postgresql/11/main/postgresql.conf 
#for pgbackrest
archive_command = 'pgbackrest --stanza=demo archive-push %p'
archive_mode = on
listen_addresses = '*'
log_line_prefix = ''
max_wal_senders = 3
wal_level = replica
postgres@hqdc039:~$
```

**archive_command**
這個因為要帶stanza 名稱進來，所以**需要跟pgbackrest.conf 裡面定義的名稱一致**

**max_wal_senders**
簡單說就是定義在抄寫wal 的時候，可以同時抄給幾台
這是postgresql 的說明
```
the maximum number of simultaneously running WAL sender processes
```

**wal_level**
有三種等級，參考 https://blog.csdn.net/pg_hgdb/article/details/78666719

* minimal --不能通过基础备份和wal日志恢复数据库。
* replica = 9.6版本以前的archive和hot_standby  --该级别支持wal归档和复制。
* logical --在replica级别的基础上添加了支持逻辑解码所需的信息。

設定完成後，重起 postgresql 就可以了

#### 建立 stanza
stanza 這名詞我是第一次聽到，直接翻譯就是 **""（詩的）節，段"**

https://dictionary.cambridge.org/zht/%E8%A9%9E%E5%85%B8/%E8%8B%B1%E8%AA%9E-%E6%BC%A2%E8%AA%9E-%E7%B9%81%E9%AB%94/stanza

在pgbackrest中，可以一次定義多個 stanza，用來備份不同的DB
這次環境很簡單，所以就只有設定一個
依照上面的pgbackrest.conf 內容設定好了，需要先建立這個stanza
指令:
```
postgres@hqdc039:~$ pgbackrest --stanza=demo stanza-create --log-level-console=detail
2019-09-04 16:21:40.700 P00   INFO: stanza-create command begin 2.16: --log-level-console=detail --pg1-path=/database/11/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-path=/var/lib/pgbackrest --stanza=demo
2019-09-04 16:21:41.525 P00   INFO: stanza-create command end: completed successfully (825ms)
postgres@hqdc039:~$ 
```

接著就可以執行備份了

#### backup

備份指令也很簡單，要注意的是，如果不帶參數，pgbackrest 會自行決定要用incremental或者是 full backup
```
postgres@hqdc039:~$ pgbackrest --stanza=demo  --log-level-console=detail backup
2019-09-04 16:41:17.458 P00   INFO: backup command begin 2.16: --log-level-console=detail --pg1-path=/database/11/main --repo1-cipher-pass=<redacted> --repo1-cipher-type=aes-256-cbc --repo1-path=/var/lib/pgbackrest --repo1-retention-full=2 --stanza=demo
2019-09-04 16:41:17.697 P00   INFO: last backup label = 20190904-134245F, version = 2.16
2019-09-04 16:41:18.607 P00   INFO: execute non-exclusive pg_start_backup() with label "pgBackRest backup started at 2019-09-04 16:41:17": backup begins after the next regular checkpoint completes
2019-09-04 16:41:19.008 P00   INFO: backup start archive = 000000100000000E0000004A, lsn = E/4A000028
WARN: a timeline switch has occurred since the last backup, enabling delta checksum
2019-09-04 16:41:21.213 P01 DETAIL: match file from prior backup /database/11/main/base/51435/51488 (546.3MB, 20%) checksum 5eb4f73d9b1c535ebfdfb622d930dade87e23786
2019-09-04 16:41:21.821 P01 DETAIL: match file from prior backup /database/11/main/base/51435/51460 (455.3MB, 37%) checksum aa74bba2bea8823789ad4194e4574b44a020271a
...
...
...

2019-09-04 16:41:24.827 P01 DETAIL: match file from prior backup /database/11/main/PG_VERSION (3B, 100%) checksum dd71038f3463f511ee7403dbcbc87195302d891c
2019-09-04 16:41:24.835 P01   INFO: backup file /database/11/main/base/13125/51569 (0B, 100%)
2019-09-04 16:41:24.860 P00   INFO: incr backup size = 2.5GB
2019-09-04 16:41:24.860 P00   INFO: execute non-exclusive pg_stop_backup() and wait for all WAL segments to archive
2019-09-04 16:41:24.961 P00   INFO: backup stop archive = 000000100000000E0000004A, lsn = E/4A000130
2019-09-04 16:41:24.964 P00 DETAIL: wrote 'pg_data/backup_label' file returned from pg_stop_backup()
2019-09-04 16:41:25.155 P00   INFO: new backup label = 20190904-134245F_20190904-164117I
2019-09-04 16:41:25.194 P00   INFO: backup command end: completed successfully (7736ms)
2019-09-04 16:41:25.194 P00   INFO: expire command begin
2019-09-04 16:41:25.214 P00   INFO: full backup total < 2 - using oldest full backup for 11-1 archive retention
2019-09-04 16:41:25.214 P00 DETAIL: archive retention on backup 20190904-134245F, archiveId = 11-1, start = 0000000D0000000E00000048
2019-09-04 16:41:25.215 P00 DETAIL: no archive to remove, archiveId = 11-1
2019-09-04 16:41:25.215 P00   INFO: expire command end: completed successfully (21ms)
postgres@hqdc039:~$ 
```

執行後，可以檢查一下
```
postgres@hqdc039:~$ pgbackrest --stanza=demo  --log-level-console=info info
stanza: demo
    status: ok
    cipher: aes-256-cbc

    db (current)
        wal archive min/max (11-1): 0000000D0000000E00000048/000000100000000E0000004A

        full backup: 20190904-134245F
            timestamp start/stop: 2019-09-04 13:42:45 / 2019-09-04 13:48:54
            wal start/stop: 0000000D0000000E00000048 / 0000000D0000000E00000048
            database size: 2.6GB, backup size: 2.6GB
            repository size: 547MB, repository backup size: 547MB

        incr backup: 20190904-134245F_20190904-164117I
            timestamp start/stop: 2019-09-04 16:41:17 / 2019-09-04 16:41:25
            wal start/stop: 000000100000000E0000004A / 000000100000000E0000004A
            database size: 2.6GB, backup size: 2.2MB
            repository size: 547MB, repository backup size: 220.4KB
            backup reference list: 20190904-134245F
postgres@hqdc039:~$
```

#### 還原測試
pgbackrest 的還原因為透過WAL ，所以有一些觀念需要先釐清
本來我的測試是先做好備份，然後直接砍掉 DB，接著再用restore還原
卻發現砍掉的DB居然沒有回來..
後來看了這一篇，然後又研究了一下 **Point-in-Time Recovery**
https://github.com/pgbackrest/pgbackrest/issues/800
才找到正確的用法

先把DB整個砍掉
```
postgres@hqdc034:/zp/database$ time dropdb demo
```
然後停止postgresql 服務
```
postgres@hqdc034:/zp/database$ pg_ctlcluster 10 main stop
  sudo systemctl stop postgresql@10-main
```
接著就可以來下指令還原了
```
postgres@hqdc034:/zp/database$ time pgbackrest --stanza=demo --log-level-console=info --delta --type=time "--target=2019-09-05 11:00:00.268248+08" --target-action=promote restore
2019-09-05 11:15:57.480 P00   INFO: restore command begin 2.13: --delta --log-level-console=info --pg1-path=/zp/database/10/main --process-max=4 --repo1-path=/var/lib/pgbackrest --stanza=demo --target="2019-09-05 11:00:00.268248+08" --target-action=promote --type=time
2019-09-05 11:15:57.624 P00   INFO: restore backup set 20190905-111109F
2019-09-05 11:15:57.947 P00   INFO: remove invalid files/paths/links from /zp/database/10/main
2019-09-05 11:16:00.440 P01   INFO: restore file /zp/database/10/main/global/pg_control.pgbackrest.tmp (8KB, 99%) checksum e253f9d706ac59e1ec0408ba477d1d5bac41b20f
2019-09-05 11:16:00.791 P00   INFO: write /zp/database/10/main/recovery.conf
2019-09-05 11:16:00.797 P00   INFO: restore global/pg_control (performed last to ensure aborted restores cannot be started)
2019-09-05 11:16:00.800 P00   INFO: restore command end: completed successfully (3321ms)

real	0m3.328s
user	0m6.896s
sys	0m0.832s
```
在DB目錄中，會產生一個recovery.conf，這是用來給 postgresql 看的檔案，裡面會紀錄怎麼還原、以及還原的類型、時間點，這邊指定要恢復到 ***2019-09-05 11:00:00.268248+08***
這個時間格式要看資料庫的設定，可以藉由以下指令得到
```
postgres@hqdc034:/zp/database$ psql -Atc "select current_timestamp"
2019-09-05 11:14:27.268248+08
```
檢查一下recovery.conf
```
postgres@hqdc034:/zp/database$ cat /zp/database/10/main/recovery.conf 
restore_command = 'pgbackrest --log-level-console=info --stanza=demo archive-get %f "%p"'
recovery_target_time = '2019-09-05 11:00:00.268248+08'
recovery_target_action = 'promote'
```
重新啟動postgresql 然後看看被砍掉的demo DB有沒有回來
```
postgres@hqdc034:/zp/database$ pg_ctlcluster 10 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@10-main
postgres@hqdc034:/zp/database$ psql 
psql (10.8 (Ubuntu 10.8-1.pgdg14.04+1))
Type "help" for help.

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 demo      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

postgres=# \q
postgres@hqdc034:/zp/database$ 
```

很好， demo DB 有順利的還原回來了，先暫時測試到這邊。接下來要來玩postgresql on zfs

後續如果想更深入測試 pgbackrest，可以試試看異機備份還原。


