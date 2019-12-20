---
title: "[筆記] 測試 postgresql 的pg_prewarm 對效能的影響 / test pg_prewarm in postgresql 11"
date: 2019-12-20T14:31:42+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-9.jpg
tags: ['postgresql']
author: "Eric Chang"
keywords:
  - postgresql
  - pg_prewarm 
---

老闆提到想要把新系統的 postgresql 資料庫都撈到記憶體裡面

但是否決了我提出的ramdisk 作法(因為當機的話，資料就沒了)

在找資料的時候，發現了這個postgresql 的 pg_prewarm extension

好像有點意思？就來測試看看吧！

只是目前還不知道該怎麼解讀測試的數據就是了...

幹！林北真的不是 DBA 啦 =.=

<!--more-->

安裝系統、postgresql 資料庫什麼的就不提了，那不是這次的重點

#### 修改 postgresql.conf

編輯postgresql.conf，開啟平行處理以及設定可用記憶體容量

這台測試機的環境是一台三代i7 , 24G RAM , 240G SSD，安裝debian 10(buster)

```
# load libiriaes
# 其實這次不會用到pg_stat_statements ，不過出於習慣，還是加入開機自動載入吧
shared_preload_libraries = 'pg_stat_statements'

#------------------------------------------------------------------------------
# CUSTOMIZED OPTIONS
#------------------------------------------------------------------------------

max_connections = 20
shared_buffers = 6GB
effective_cache_size = 18GB
maintenance_work_mem = 1536MB
checkpoint_completion_target = 0.7
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 78643kB
min_wal_size = 1GB
max_wal_size = 2GB
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
```

重新啟動postgresql ，準備開始測試囉！

轉換成 postgres 身份後，進入 psql

#### 建立測試資料庫

```
postgres=# create database test;
CREATE DATABASE
postgres=#
```

#### 連接測試資料庫、建立pg_prewarm extension
```
postgres=# \c test ;
You are now connected to database "test" as user "postgres".
test=# CREATE EXTENSION pg_prewarm;
CREATE EXTENSION
test=# 
```

#### 建立測試資料表，塞入500萬筆資料

```
test=# \timing
Timing is on.
test=# CREATE TABLE test_tbl AS
SELECT floor(random() * (9923123) + 1)::int FROM generate_series(1, 5000000) AS id;
SELECT 5000000
Time: 2940.602 ms (00:02.941)
test=#
```
#### 檢查看看剛剛建立的table 用了多少空間

哎呀，看起來用得不多啊
```
test=# SELECT pg_size_pretty(pg_relation_size('test_tbl'));
 173 MB
```
**玩大一點，塞個一億筆資料好了**

```
test=# drop table test_tbl;
Time: 0.361 ms
test=# CREATE TABLE test_tbl AS
SELECT floor(random() * (99343) + 1)::int FROM generate_series(1, 100000000) AS id;
SELECT 100000000
Time: 6321.415 ms (00:06.321)

test=# SELECT pg_size_pretty(pg_relation_size('test_tbl'));
pg_size_pretty | 3457 MB

Time: 0.589 ms
test=#

```

好，現在資料庫長到3457MB了

先來執行一些初步的取得基本數據

```
test=# explain (analyze,buffers) select count(*) from test_tbl;
                                   QUERY PLAN
------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=755978.52..755978.53 rows=1 width=8) (actual time=3331.917..3331.918 rows=1 loops=1)
   Buffers: shared hit=160 read=442318
   ->  Gather  (cost=755978.10..755978.51 rows=4 width=8) (actual time=3331.876..3333.674 rows=5 loops=1)
         Workers Planned: 4
         Workers Launched: 4
         Buffers: shared hit=160 read=442318
         ->  Partial Aggregate  (cost=754978.10..754978.11 rows=1 width=8) (actual time=3329.279..3329.280 rows=1 loops=5)
               Buffers: shared hit=160 read=442318
               ->  Parallel Seq Scan on test_tbl  (cost=0.00..692478.08 rows=25000008 width=0) (actual time=0.029..1924.601 rows=20000000 loops=5)
                     Buffers: shared hit=160 read=442318
 Planning Time: 0.040 ms
 Execution Time: 3333.729 ms
(12 rows)

(END)

```

可以看到打中buffer 的部份其實很少，只有 160 ，大部分都是讀進去buffer (442318)

來看看 buffer 的使用狀況
```
test=# CREATE EXTENSION pg_buffercache;
CREATE EXTENSION
test=# select c.relname,pg_size_pretty(count(*) * 8192) as buffered, 
test-#         round(100.0 * count(*) / ( 
test(#            select setting from pg_settings 
test(#            where name='shared_buffers')::integer,1)
test-#         as buffer_percent, 
test-#         round(100.0*count(*)*8192 / pg_table_size(c.oid),1) as percent_of_relation
test-# from pg_class c inner join pg_buffercache b on b.relfilenode = c.relfilenode inner 
test-# join pg_database d on ( b.reldatabase =d.oid and d.datname =current_database()) 
test-# group by c.oid,c.relname order by 3 desc limit 10;
   relname    |  buffered  | buffer_percent | percent_of_relation 
--------------+------------+----------------+---------------------
 test_tbl     | 18 MB      |            0.3 |                 0.5
 pg_am        | 8192 bytes |            0.0 |                20.0
 pg_index     | 24 kB      |            0.0 |                37.5
 pg_amproc    | 32 kB      |            0.0 |                50.0
 pg_cast      | 16 kB      |            0.0 |                33.3
 pg_depend    | 64 kB      |            0.0 |                13.3
 pg_amop      | 48 kB      |            0.0 |                54.5
 pg_namespace | 8192 bytes |            0.0 |                20.0
 pg_opclass   | 16 kB      |            0.0 |                28.6
 pg_aggregate | 8192 bytes |            0.0 |                16.7
(10 rows)

Time: 148.719 ms
test=# 
```

可以看到這個 test_tbl 只有0.5% 被撈到shared_buffers 裡面

接下來就把這個table全部推到shared_buffers 裡面去

```
test=# select pg_prewarm('test_tbl','buffer');
 pg_prewarm 
------------
     442478
(1 row)

Time: 1938.043 ms (00:01.938)
test=#
```

然後再來看一次shared_buffers的使用狀況
```
test=# select c.relname,pg_size_pretty(count(*) * 8192) as buffered, 
        round(100.0 * count(*) / ( 
           select setting from pg_settings 
           where name='shared_buffers')::integer,1)
        as buffer_percent, 
        round(100.0*count(*)*8192 / pg_table_size(c.oid),1) as percent_of_relation
from pg_class c inner join pg_buffercache b on b.relfilenode = c.relfilenode inner 
join pg_database d on ( b.reldatabase =d.oid and d.datname =current_database()) 
group by c.oid,c.relname order by 3 desc limit 10;
   relname    |  buffered  | buffer_percent | percent_of_relation 
--------------+------------+----------------+---------------------
 test_tbl     | 3457 MB    |           56.3 |               100.0
 pg_am        | 8192 bytes |            0.0 |                20.0
 pg_index     | 24 kB      |            0.0 |                37.5
 pg_amproc    | 32 kB      |            0.0 |                50.0
 pg_cast      | 16 kB      |            0.0 |                33.3
 pg_depend    | 64 kB      |            0.0 |                13.3
 pg_amop      | 48 kB      |            0.0 |                54.5
 pg_namespace | 8192 bytes |            0.0 |                20.0
 pg_opclass   | 16 kB      |            0.0 |                28.6
 pg_aggregate | 8192 bytes |            0.0 |                16.7
(10 rows)

Time: 2778.354 ms (00:02.778)
test=# 
```

OK ，可以看到 test_tbl 已經通通被載入 shared_buffers 中

**buffered 表示表格被載入shared_buffers的大小**

**buffer_percent 表示這個表格佔用多少shared_buffers 的比例**

**percent_of_relation 表示這個表格有多少比例被載入 shared_buffers**


再來跑一次explain看看狀況

```
test=# explain (analyze,buffers) select count(*) from test_tbl;
Time: 3551.785 ms (00:03.552)
 Finalize Aggregate  (cost=755978.52..755978.53 rows=1 width=8) (actual time=3427.286..3427.287 rows=1 loops=1)
   Buffers: shared hit=442478
   ->  Gather  (cost=755978.10..755978.51 rows=4 width=8) (actual time=3427.215..3551.326 rows=5 loops=1)
         Workers Planned: 4
         Workers Launched: 4
         Buffers: shared hit=442478
         ->  Partial Aggregate  (cost=754978.10..754978.11 rows=1 width=8) (actual time=3423.659..3423.659 rows=1 loops=5)
               Buffers: shared hit=442478
               ->  Parallel Seq Scan on test_tbl  (cost=0.00..692478.08 rows=25000008 width=0) (actual time=0.017..1976.744 rows=20000000 loops=5)
                     Buffers: shared hit=442478
 Planning Time: 0.039 ms
 Execution Time: 3551.365 ms
(12 rows)

```

這邊就可以看到都是從buffer 讀出來所以 hit=442478

看樣子表格還是太小，所以沒有完全發揮？那再來把表格加大！

先重開一次 postgresql 清除buffer

然後重新建立表格
```
test=# drop table test_tbl;
DROP TABLE
Time: 297.493 ms
test=# CREATE TABLE test_tbl AS
test-# SELECT floor(random() * (993343) + 1)::int FROM generate_series(1, 300000000) AS id;
SELECT 300000000
Time: 290660.607 ms (04:50.661)
test=# 
```

一樣，看看用了多少容量
```
test=# SELECT pg_size_pretty(pg_relation_size('test_tbl'));
 pg_size_pretty 
----------------
 10 GB
(1 row)

Time: 0.474 ms
test=# 
```

哇哈哈，用了10G ，這次還不撐爆你！

跑explain 看看狀況


```
test=# explain (analyze,buffers) select count(*) from test_tbl;
Time: 22909.065 ms (00:22.909)

                                              QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=2265934.72..2265934.73 rows=1 width=8) (actual time=22906.045..22906.045 rows=1 loops=1)
   Buffers: shared hit=2080 read=1325354 dirtied=1295425 written=1295265
   ->  Gather  (cost=2265934.30..2265934.71 rows=4 width=8) (actual time=22905.997..22908.522 rows=5 loops=1)
         Workers Planned: 4
         Workers Launched: 4
         Buffers: shared hit=2080 read=1325354 dirtied=1295425 written=1295265
         ->  Partial Aggregate  (cost=2264934.30..2264934.31 rows=1 width=8) (actual time=22903.473..22903.474 rows=1 loops=5)
               Buffers: shared hit=2080 read=1325354 dirtied=1295425 written=1295265
               ->  Parallel Seq Scan on test_tbl  (cost=0.00..2077434.24 rows=75000024 width=0) (actual time=0.040..18374.277 rows=60000000 loops=5)
                     Buffers: shared hit=2080 read=1325354 dirtied=1295425 written=1295265
 Planning Time: 0.094 ms
 Execution Time: 22908.571 ms
(12 rows)

```

看一下現在 shared_buffers 使用狀況

可以看到這個 test_tbl 幾乎沒被放入 shared_buffers 中

```
test=# select c.relname,pg_size_pretty(count(*) * 8192) as buffered, 
        round(100.0 * count(*) / ( 
           select setting from pg_settings 
           where name='shared_buffers')::integer,1)
        as buffer_percent, 
        round(100.0*count(*)*8192 / pg_table_size(c.oid),1) as percent_of_relation
from pg_class c inner join pg_buffercache b on b.relfilenode = c.relfilenode inner 
join pg_database d on ( b.reldatabase =d.oid and d.datname =current_database()) 
group by c.oid,c.relname order by 3 desc limit 10;
   relname    |  buffered  | buffer_percent | percent_of_relation 
--------------+------------+----------------+---------------------
 test_tbl     | 18 MB      |            0.3 |                 0.2
 pg_am        | 8192 bytes |            0.0 |                20.0
 pg_index     | 24 kB      |            0.0 |                37.5
 pg_amproc    | 32 kB      |            0.0 |                50.0
 pg_cast      | 16 kB      |            0.0 |                33.3
 pg_depend    | 64 kB      |            0.0 |                13.3
 pg_amop      | 48 kB      |            0.0 |                54.5
 pg_namespace | 8192 bytes |            0.0 |                20.0
 pg_opclass   | 16 kB      |            0.0 |                28.6
 pg_aggregate | 8192 bytes |            0.0 |                16.7
(10 rows)

Time: 163.936 ms
test=# 
```

強制把test_tbl 全部塞進 shared_buffers

```
test=# select pg_prewarm('test_tbl','buffer');
 pg_prewarm 
------------
    1327434
(1 row)

Time: 7472.805 ms (00:07.473)
test=# 
```

確認一下test_tbl 有沒有被整個塞進去

```
test=# select c.relname,pg_size_pretty(count(*) * 8192) as buffered, 
        round(100.0 * count(*) / ( 
           select setting from pg_settings 
           where name='shared_buffers')::integer,1)
        as buffer_percent, 
        round(100.0*count(*)*8192 / pg_table_size(c.oid),1) as percent_of_relation
from pg_class c inner join pg_buffercache b on b.relfilenode = c.relfilenode inner 
join pg_database d on ( b.reldatabase =d.oid and d.datname =current_database()) 
group by c.oid,c.relname order by 3 desc limit 10;
   relname    |  buffered  | buffer_percent | percent_of_relation 
--------------+------------+----------------+---------------------
 test_tbl     | 6142 MB    |          100.0 |                59.2
 pg_am        | 8192 bytes |            0.0 |                20.0
 pg_index     | 24 kB      |            0.0 |                37.5
 pg_amproc    | 32 kB      |            0.0 |                50.0
 pg_cast      | 16 kB      |            0.0 |                33.3
 pg_depend    | 24 kB      |            0.0 |                 5.0
 pg_amop      | 40 kB      |            0.0 |                45.5
 pg_namespace | 8192 bytes |            0.0 |                20.0
 pg_opclass   | 16 kB      |            0.0 |                28.6
 pg_aggregate | 8192 bytes |            0.0 |                16.7
(10 rows)

Time: 4985.366 ms (00:04.985)
test=#
```

GOOD ！ let's do explain again !

```
test=# explain (analyze,buffers) select count(*) from test_tbl;
Time: 11451.188 ms (00:11.451)
                             QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Finalize Aggregate  (cost=2265934.72..2265934.73 rows=1 width=8) (actual time=11231.664..11231.664 rows=1 loops=1)
   Buffers: shared hit=785963 read=541471
   ->  Gather  (cost=2265934.30..2265934.71 rows=4 width=8) (actual time=11231.606..11450.719 rows=5 loops=1)
         Workers Planned: 4
         Workers Launched: 4
         Buffers: shared hit=785963 read=541471
         ->  Partial Aggregate  (cost=2264934.30..2264934.31 rows=1 width=8) (actual time=11228.829..11228.830 rows=1 loops=5)
               Buffers: shared hit=785963 read=541471
               ->  Parallel Seq Scan on test_tbl  (cost=0.00..2077434.24 rows=75000024 width=0) (actual time=0.037..6414.711 rows=60000000 loops=5)
                     Buffers: shared hit=785963 read=541471
 Planning Time: 0.039 ms
 Execution Time: 11450.781 ms
(12 rows)
```

確認一下，果然大部分都打到cache 了，但是因為shared_buffers 不夠大，所以還會從磁碟讀取一部分

而且時間也比之前還沒塞進shared_buffers 的時候要快了不少

22908.571 --> 11450.781 ms

***

從這次的測試看來，我想如果有足夠大的記憶體，能夠把資料表都塞入shared_buffers 中

應該可以帶來不錯的效能增幅！

