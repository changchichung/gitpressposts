---
title: "[筆記] 用zfs的snapshot 快照功能來做 postgresql 的備份還原 / Postgresql Backup Restore Using Zfs Snapshot"
date: 2019-09-06T10:42:11+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-5.jpg"
categories: ['筆記']
tags: ['postgresql','zfs','backup','restore']
author: "Eric Chang"
keywords:
  - postgresql
  - zfs
  - backup
  - restore
---

前面測試了用pgbarman / pgbackrest 來備份 postgresql

這次改從system file level 來下手

採用zfs 的快照來備份、還原postgresql 資料庫


<!--more-->
### 建立測試資料庫、TABLE、snapshot

#### 資料庫現況
只有系統預設的DB，沒有其他多的東西
```
postgres@hqdc034:~$ psql -c '\l'
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres@hqdc034:~$ du -sh /zp/database/10/main/
232M	/zp/database/10/main/
```

#### 建立第一次的快照
```
2019-09-06 09:03:46 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
no datasets available
2019-09-06 09:03:53 [changch@hqdc034 ~]$ sudo zfs snapshot zp/database@init_db_no_demo
2019-09-06 09:04:09 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                          USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo      0      -   231M  -
2019-09-06 09:04:15 [changch@hqdc034 ~]$
```

#### 建立、倒回測試資料庫 demo

```
postgres@hqdc034:~$ createdb demo
postgres@hqdc034:~$ psql demo < /home/changch/Downloads/demo.sql 
SET
SET
略...
```

再檢查一次資料庫的狀況，看到 demo DB出現了，資料庫目錄也變大了
```
postgres@hqdc034:~$ psql -c '\l'
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

postgres@hqdc034:~$ du -sh /zp/database/10/main/
2.1G	/zp/database/10/main/
postgres@hqdc034:~$ 
```

#### 建立第二次快照
這次的快照，將包含剛剛倒回的 demo DB，但是不包含等下才要建立的測試 table
```
2019-09-06 09:16:01 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                               USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo        250K      -   231M  -
zp/database@demo_db_just_restore      0      -  2.08G  -
2019-09-06 09:16:04 [changch@hqdc034 ~]$
```

#### 建立測試 table
```
postgres@hqdc034:~$ psql -c 'create table test ( a int, b varchar(50) );'
CREATE TABLE
postgres@hqdc034:~$
```
#### 建立第三次快照
這次快照，只有建立 test table ，但是裡面沒有資料
```
2019-09-06 09:18:34 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data      0      -  2.08G  -
2019-09-06 09:18:36 [changch@hqdc034 ~]$
```
#### 在test table 插入100萬筆資料
```
postgres@hqdc034:~$ psql -c 'with aa as ( select * from generate_series (1,1000000) a ) insert into test select aa.a, md5(aa.a::varchar) from aa;'
INSERT 0 1000000
postgres@hqdc034:~$ psql -c 'select count(*) from test;'
  count  
---------
 1000000
(1 row)

postgres@hqdc034:~$
```
#### 建立第四次快照
test table 內有 1000000 筆資料
```
2019-09-06 09:18:36 [changch@hqdc034 ~]$ sudo zfs snapshot zp/database@demo_db_test_table_with_1M_rows
2019-09-06 09:21:08 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data   116K      -  2.08G  -
zp/database@demo_db_test_table_with_1M_rows            0      -  2.15G  -
2019-09-06 09:21:09 [changch@hqdc034 ~]$
```

#### 再次插入 100萬筆資料
```
postgres@hqdc034:~$ time psql -c 'with aa as ( select * from generate_series (1,1000000) a ) insert into test select aa.a, md5(aa.a::varchar) from aa;'
INSERT 0 1000000

real	0m4.276s
user	0m0.020s
sys	0m0.012s
postgres@hqdc034:~$ psql -c 'select count(*) from test;'
  count  
---------
 2000000
(1 row)

postgres@hqdc034:~$
```

#### 建立第五次快照
現在 test table 有 200萬筆資料了
```
2019-09-06 09:21:09 [changch@hqdc034 ~]$ sudo zfs snapshot zp/database@demo_db_test_table_with_2M_rows
2019-09-06 09:22:29 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data   116K      -  2.08G  -
zp/database@demo_db_test_table_with_1M_rows         218K      -  2.15G  -
zp/database@demo_db_test_table_with_2M_rows            0      -  2.23G  -
2019-09-06 09:22:30 [changch@hqdc034 ~]$
```
#### 玩大點，直接湊滿1000萬筆資料好了
```
postgres@hqdc034:~$ time psql -c 'with aa as ( select * from generate_series (1,8000000) a ) insert into test select aa.a, md5(aa.a::varchar) from aa;'
INSERT 0 8000000

real	0m32.172s
user	0m0.024s
sys	0m0.008s
postgres@hqdc034:~$ psql -c 'select count(*) from test;'
  count   
----------
 10000000
(1 row)
postgres@hqdc034:~$
```

#### 建立第六次快照
10M rows in test table
```
2019-09-06 09:22:30 [changch@hqdc034 ~]$ sudo zfs snapshot zp/database@demo_db_test_table_with_10M_rows
2019-09-06 09:25:18 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data   116K      -  2.08G  -
zp/database@demo_db_test_table_with_1M_rows         218K      -  2.15G  -
zp/database@demo_db_test_table_with_2M_rows         530K      -  2.23G  -
zp/database@demo_db_test_table_with_10M_rows        163K      -  2.97G  -
2019-09-06 09:25:21 [changch@hqdc034 ~]$
```

到1000萬筆資料為止，現在資料庫大小是這樣
```
postgres@hqdc034:~$ du -sh /zp/database/10/main/
3.0G	/zp/database/10/main/
postgres@hqdc034:~$
```


***
### 還原測試

最後一次做快照的時候，demo DB 裡面有一千萬筆資料，現在來砍掉500萬筆
```
postgres@hqdc034:~$ time psql -c 'delete from test where a > 5000000;'
DELETE 3000000

real	0m7.844s
user	0m0.024s
sys	0m0.004s

postgres@hqdc034:~$ time psql -c 'select count(*) from test;'
  count  
---------
 7000000
(1 row)


real	0m0.268s
user	0m0.024s
sys	0m0.004s
postgres@hqdc034:~$
```
咦，怪怪的，為什麼只有砍掉300萬筆？
好，這邊先不管，等等正好來驗證restore的狀況 

假設剛剛這個刪除是錯誤的動作，我要回到1000萬資料的狀態，就可以用zfs rollback 來達成

#### 第一次還原
目標是還原到包含1000萬筆資料的狀態(現在是700萬筆)

```
2019-09-06 09:25:21 [changch@hqdc034 ~]$ sudo service postgresql stop
 * Stopping PostgreSQL 10 database server      [ OK ] 
2019-09-06 10:14:12 [changch@hqdc034 ~]$ sudo zfs rollback -r zp/database@demo_db_test_table_with_10M_rows
2019-09-06 10:14:28 [changch@hqdc034 ~]$ sudo service postgresql start
 * Starting PostgreSQL 10 database server      [ OK ]
2019-09-06 10:14:57 [changch@hqdc034 ~]$ 
```
檢查一下
```
postgres@hqdc034:~$ time psql -c 'select count(*) from test;'
  count   
----------
 10000000
(1 row)


real	0m5.019s
user	0m0.040s
sys	0m0.008s
postgres@hqdc034:~$
```
沒錯，又回到1000萬筆資料的狀態了

要注意的是，如果回到更之前的狀態，在該狀態之後的快照將會被清除，除非你先做clone
比如我現在要回到 200萬筆的狀態，那1000萬筆資料的快照就會被刪除
```
2019-09-06 10:17:32 [changch@hqdc034 ~]$ sudo service postgresql stop
 * Stopping PostgreSQL 10 database server         [ OK ] 
2019-09-06 10:18:50 [changch@hqdc034 ~]$ sudo zfs rollback -r zp/database@demo_db_test_table_with_2M_rows
2019-09-06 10:18:57 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data   116K      -  2.08G  -
zp/database@demo_db_test_table_with_1M_rows         218K      -  2.15G  -
zp/database@demo_db_test_table_with_2M_rows            0      -  2.23G  -
2019-09-06 10:19:04 [changch@hqdc034 ~]$ sudo service postgresql start
 * Starting PostgreSQL 10 database server        [ OK ] 
2019-09-06 10:19:17 [changch@hqdc034 ~]$

postgres@hqdc034:~$ time psql -c 'select count(*) from test;'
  count  
---------
 2000000
(1 row)

real	0m0.175s
user	0m0.024s
sys	0m0.008s
postgres@hqdc034:~$ 
```

啊，我剛剛應該先clone的....
沒關係，我們再做一次，新增800萬筆資料，湊齊1000萬筆，然後快照
```
postgres@hqdc034:~$ time psql -c 'with aa as ( select * from generate_series (1,8000000) a ) insert into test select aa.a, md5(aa.a::varchar) from aa;'
INSERT 0 8000000

real	0m35.662s
user	0m0.048s
sys	0m0.004s
postgres@hqdc034:~$ time psql -c 'select count(*) from test;'
  count   
----------
 10000000
(1 row)

real	0m5.259s
user	0m0.024s
sys	0m0.008s
postgres@hqdc034:~$ 
```
做快照
```
2019-09-06 10:19:17 [changch@hqdc034 ~]$ sudo zfs snapshot zp/database@demo_db_test_table_with_10M_rows
2019-09-06 10:22:59 [changch@hqdc034 ~]$ sudo zfs list -t snapshot
NAME                                                USED  AVAIL  REFER  MOUNTPOINT
zp/database@init_db_no_demo                         250K      -   231M  -
zp/database@demo_db_just_restore                    138K      -  2.08G  -
zp/database@demo_db_create_test_table_but_no_data   116K      -  2.08G  -
zp/database@demo_db_test_table_with_1M_rows         218K      -  2.15G  -
zp/database@demo_db_test_table_with_2M_rows        56.4M      -  2.23G  -
zp/database@demo_db_test_table_with_10M_rows           0      -  1.81G  -
2019-09-06 10:23:02 [changch@hqdc034 ~]$
```
接著來測試看看 clone snapshot，這是基本的說明
```
Clones can only be created from a snapshot and a snapshot can not 
be deleted until you delete the clone that is based on this snapshot. 
To create a clone, use the zfs clone command.
```
clone 會做出一份跟clone來源一模一樣的資料，在快照模式下，資料是唯讀的，clone出來後，就可以做異動。但是不能刪除clone來源的快照，會提示錯誤。
```
2019-09-06 10:28:31 [changch@hqdc034 ~]$ sudo zfs clone zp/database@demo_db_test_table_with_10M_rows zp/database/clone_with_10M_rows
2019-09-06 10:29:21 [changch@hqdc034 ~]$ sudo zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
zp                               3.08G   231G    22K  /zp
zp/database                      3.08G   231G  1.88G  /zp/database
zp/database/clone_with_10M_rows      0   231G  1.81G  /zp/database/clone_with_10M_rows
2019-09-06 10:29:26 [changch@hqdc034 ~]$
```
可以看到做了clone之後，多了一個 zfs dataset
試試看把資料庫路徑直接改到這個新做的dataset 看看能不能啟動資料庫
修改 /etc/postgresql/10/main/postgresql.conf，然後重起postgresql


```
#data_directory = '/var/lib/postgresql/10/main'     # use data in another directory
#data_directory = '/zp/database/10/main'
data_directory = '/zp/database/clone_with_10M_rows/10/main'
```

**啟動有比較久一點** 而且好像沒成功啟動
```
2019-09-06 10:32:27 [changch@hqdc034 ~]$ sudo service postgresql restart
 * Restarting PostgreSQL 10 database server       [ OK ]
2019-09-06 10:33:37 [changch@hqdc034 ~]$ 
2019-09-06 10:33:37 [changch@hqdc034 ~]$ sudo netstat -antlp |grep 5432
```
而且在 syslog & postgresql log 中看不到什麼異常，怪了！
而且再啟動一次就好了？

再來測試一次看看
```
2019-09-06 10:37:22 [changch@hqdc034 ~]$ sudo service postgresql stop
 * Stopping PostgreSQL 10 database server        [ OK ] 
2019-09-06 10:38:03 [changch@hqdc034 ~]$ sudo zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
zp                               3.24G   231G    22K  /zp
zp/database                      3.24G   231G  1.88G  /zp/database
zp/database/clone_with_10M_rows   165M   231G  1.88G  /zp/database/clone_with_10M_rows
2019-09-06 10:38:13 [changch@hqdc034 ~]$ sudo zfs destroy zp/database/clone_with_10M_rows
2019-09-06 10:38:21 [changch@hqdc034 ~]$ sudo zfs clone zp/database@demo_db_test_table_with_10M_rows zp/database/clone_with_10M_rows
2019-09-06 10:38:32 [changch@hqdc034 ~]$ sudo zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
zp                               3.08G   231G    22K  /zp
zp/database                      3.08G   231G  1.88G  /zp/database
zp/database/clone_with_10M_rows      0   231G  1.81G  /zp/database/clone_with_10M_rows
2019-09-06 10:38:35 [changch@hqdc034 ~]$ sudo service postgresql start^C
2019-09-06 10:38:44 [changch@hqdc034 ~]$ sudo zfs list
NAME                              USED  AVAIL  REFER  MOUNTPOINT
zp                               3.08G   231G    22K  /zp
zp/database                      3.08G   231G  1.88G  /zp/database
zp/database/clone_with_10M_rows      0   231G  1.81G  /zp/database/clone_with_10M_rows
2019-09-06 10:38:45 [changch@hqdc034 ~]$ sudo service postgresql start
 * Starting PostgreSQL 10 database server               [ OK ] 
2019-09-06 10:39:04 [changch@hqdc034 ~]$ netstat -antlp |grep 5432
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      -               
tcp6       0      0 :::5432                 :::*                    LISTEN      -               
2019-09-06 10:39:13 [changch@hqdc034 ~]$ 
```
這次就沒問題？看來是我第一次下指令的時候，不該用sudo netstat -antlp 去檢查？
anyway ，回到psql 來看看內容
```
postgres@hqdc034:~$ time psql -c 'select count(*) from test;'
  count   
----------
 10000000
(1 row)


real	0m4.716s
user	0m0.028s
sys	0m0.004s
postgres@hqdc034:~$ 
```

Good ，clone 出來的果然是1000萬筆資料時的狀態

***

這次測試就先到此為止，後面再來測試zfs的 replication and send/recv
