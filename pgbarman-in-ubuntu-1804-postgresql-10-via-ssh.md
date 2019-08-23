---
title: "[筆記] 在Ubuntu 18.04 下 透過 pgbarman rsync/ssh backup 備份 postgresql 10 / backup postgresql 10 with pgbarman via ssh/rsync in ubuntu 18.04"
date: 2019-08-23T14:54:13+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-4.jpg"
categories: ['筆記']
tags: ['postgresql','pgbarman']
author: "Eric Chang"
keywords:
  - pgbarman 
  - postgresql 
---

這篇繼續講 pgbarman 透過 rsync/ssh 來備份 postgresql 資料庫的方式

<!--more-->

其實呢，透過 ssh 的方式來做備份，雖然最後有弄出來，但是我不知道到底做了什麼事才搞定

也許要重新安裝一台來測試看看

所以就簡單說一下邏輯、記得的指令、還有最後的config

##### ssh

在db server 上，讓 barman server 可以用barman 帳號透過ssh key 登入 postgres 帳號
然後在 barman server 上，讓 db server 可以用postgres帳號透過 ssh key 登入 barman 帳號

##### barman.d

在/etc/barman.d/ 底下新增一個hqs019-ssh.conf

內容如下
```
[hqs019-ssh]
description =  "hqs019 (via SSH)"
ssh_command = ssh postgres@hqs019
conninfo = host=hqs019 user=barman dbname=database_name
backup_method = rsync
reuse_backup = link
parallel_jobs = 5
archiver = on
```

##### postgresql.conf

新增跟barman有關的設定如下
```
### for barman test 
wal_level = 'archive'
archive_mode = on
archive_command = 'rsync -a %p barman@192.168.11.192:/var/lib/barman/hqs019-ssh/incoming/%f'
```

理論上這樣就可以了，但是實際上碰到很多問題

主要都是這個錯誤訊息

```
WAL archive: FAILED (please make sure WAL shipping is setup)
```

碰到這狀況，google 都會告訴你，在 postgresql.conf 裡面 archive_command 的路徑，要和 barman server 上的設定一致
![](https://i.imgur.com/3dMdytn.png)

但是我的路徑就沒有問題，還是一直跳這個錯誤

接著我又在 barman server 這邊做了這些事情

```
barman check hqs019-ssh
barman check hqs019-ssh
barman check hqs019-ssh
ssh postgres@hqs019
barman check hqs019-ssh
barman show-server hqs019-ssh |grep incoming_wals_directory
barman show-server all
barman cron
barman switch-xlog hqs019-ssh
barman check hqs019-ssh
psql -c 'SELECT version()' -U postgres -h hqs019
psql -c 'SELECT version()' -U barman -h hqs019
psql -c 'SELECT version()' -U barman -h hqs019 -d database_name 
barman check hqs019-ssh
barman check hqs019-ssh
barman bachup hqs019-ssh
barman backup hqs019-ssh
barman list-backup hqs019-ssh
df -h
barman backup hqs019-ssh
barman show-server hqs019
barman check hqs019

```
前面幾次 barman check 一直都不通過

然後在 barman cron / barman switch-xlog hqs019-ssh (其實這個動作我做過好多次)

想說確認一下psql 連接是不是正確(也的確正確無誤)

接著第一次 check 還是FAILED 喔，過沒多久我再跑一次 check 又正常了... WTF !!!

只要check 正常，接著跑 backup 應該就會很順利
```
barman@barman:~$ barman backup hqs019-ssh
Starting backup using rsync-exclusive method for server hqs019-ssh in /var/lib/barman/hqs019-ssh/base/20190823T113229
Backup start at LSN: 264/B7000028 (0000000100000264000000B7, 00000028)
This is the first backup for server hqs019-ssh
WAL segments preceding the current backup have been found:
	0000000100000264000000B5 from server hqs019-ssh has been removed
Starting backup copy via rsync/SSH for 20190823T113229 (5 jobs)
Copy done (time: 1 hour, 5 minutes, 39 seconds)
This is the first backup for server hqs019-ssh
WAL segments preceding the current backup have been found:
	0000000100000264000000B6 from server hqs019-ssh has been removed
Asking PostgreSQL server to finalize the backup.
Backup size: 132.9 GiB. Actual size on disk: 132.9 GiB (-0.00% deduplication ratio).
Backup end at LSN: 264/B7000130 (0000000100000264000000B7, 00000130)
Backup completed (start time: 2019-08-23 11:32:30.078310, elapsed time: 1 hour, 5 minutes, 43 seconds)
Processing xlog segments from file archival for hqs019-ssh
	0000000100000264000000B7
	0000000100000264000000B7.00000028.backup
barman@barman:~$
```

檢查一下狀態
```
barman@barman:~$ barman list-backup hqs019-ssh
hqs019-ssh 20190823T113229 - Thu Aug 22 20:38:13 2019 - Size: 132.9 GiB - WAL Size: 0 B (tablespaces: tablespace_a:/var/lib/postgresql/10/main/tablespace_A, tablespace_b:/var/lib/postgresql/10/main/tablespace_B)
```

然後為了驗證是不是跑增量備份，所以再執行一次 backup
```
barman@barman:~$ barman backup hqs019-ssh
Starting backup using rsync-exclusive method for server hqs019-ssh in /var/lib/barman/hqs019-ssh/base/20190823T132124
Backup start at LSN: 264/B9000028 (0000000100000264000000B9, 00000028)
Starting backup copy via rsync/SSH for 20190823T132124 (5 jobs)
Copy done (time: 5 seconds)
Asking PostgreSQL server to finalize the backup.
Backup size: 132.9 GiB. Actual size on disk: 14.2 KiB (-100.00% deduplication ratio).
Backup end at LSN: 264/B9000130 (0000000100000264000000B9, 00000130)
Backup completed (start time: 2019-08-23 13:21:24.455819, elapsed time: 9 seconds)
Processing xlog segments from file archival for hqs019-ssh
	0000000100000264000000B8
	0000000100000264000000B9
	0000000100000264000000B9.00000028.backup
barman@barman:~$
```

可以發現第一次跑了一個多小時，第二次只跑了五秒鐘 (因為資料庫根本沒異動)

到這邊雖然功能驗證沒有問題，可是不知道怎麼弄出來的，還是讓我很阿雜..

應該會再找時間來重作一台，然後順便測試看看restore


update

剛剛又做了一次測試，config 都一樣

果然要先做 barman cron/barman switch-xlog hqs019-ssh 

然後再做 barman check 就可以通過了

在文件裡面很少提到這部份，筆記一下，用ansible 去跑的時候才不會忘記

