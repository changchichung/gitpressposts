---
title: "[筆記] 在Ubuntu 18.04 下 透過 pgbarman streaming backup 備份 postgresql 10/ backup postgresql 10 with pgbarman straming backup in ubuntu 18.04"
date: 2019-08-23T13:53:40+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-4.jpg"
categories: ['筆記']
tags: ['postgresql','pgbarman']
author: "Eric Chang"
keywords:
  - pgbarman
  - postgresql 
---

很久以前就有看到這個用來備份postgresql 的 pgbarman 

https://www.pgbarman.org/

前幾天老闆在slack 上面又提到，所以這次就花了點時間來玩玩看

不過呢，雖然有弄起來，但是還真不知道有些問題是怎麼解決的...

<!--more-->

pgbarman 的備份有分兩種

streaming && rsync/SSH

原理就不說了，我一直沒搞懂 postgresql 的 streaming ..

依照官方網站上的說法，比較推薦 streaming 備份的方式

原因是，設定相對簡單，WTF !

```
On a general basis, starting from Barman 2.0, backup over streaming replication is the recommended setup for PostgreSQL 9.4 or higher
The reason why we recommend streaming backup is that, based on our experience, it is easier to setup than the traditional one
```

事實上呢，設定的確是很簡單，可是有個致命的缺點
```
Because Barman transparently makes use of pg_basebackup, features such as incremental backup, parallel backup, deduplication, and network compression are currently not available. In this case, bandwidth limitation has some restrictions - compared to the traditional method via rsync.
```

如果要做差異/增量備份， streaming backup 不能做

所以每次備份都是完整備份，也因此，barman server 需要準備很大的硬碟空間

以我測試的資料庫來說，一次備份，目前是133G ，如果一天四次，保留七天

就需要 133 x 4 x 7 = 3724G 

咦，這樣看起來，其實也還好啦 XDD

______

現在開始設定的部份
#### 設定 postgresql server
IP: 192.168.11.19
hostname: hqs019


##### 在postgresql 建立相關帳號

streaming backup 需要先在postgresql Server 上建立一個具有 superuser 權限的帳號

以及一個用來做replication 的資料庫帳號

這裡就簡單帶過

```
sudo su - postgres
psql

create user barman with login superuser login password 'barmanpassword';
CREATE ROLE stream_barman WITH REPLICATION PASSWORD 'password' LOGIN;
```

##### 鄉改 pg_hba.conf

然後修改 pg_hba.conf，加入底下兩行
```
# for barman test
host database_name barman 192.168.11.192/32 md5
host replication stream_barman 192.168.11.192/32 md5
```
當然，如果不考慮安全性問題， md5 直接改成用 trust ，可以省去一些麻煩。

##### 修改 postgresql.conf

接著修改postgresql.conf
```
### for barman test 
max_wal_senders = 5
max_replication_slots = 3
wal_level = 'archive'
archive_mode = on
```

重起 postgresql service

#### 設定barman server

IP: 192.168.11.192
hostname: barman

##### 安裝 barman

barman 在18.04 中，已經被放到標準repository 中

所以只要直接 
```
sudo apt install barman
```
就可以了


##### 設定 barman.conf

安裝完成後，在/etc/barman.d/ 底下會有兩個範例檔案
```
streaming-server.conf-template
ssh-server.conf-template
```
複製 streaming-server 檔案
```
sudo cp /etc/barman.d/streaming-server.conf-template /etc/barman.d/hqs019.conf
```
內容如下
```
[hqs019]
description = "hqs019 "
conninfo = host=192.168.11.19 user=barman dbname=database_name password=barmanpassword
streaming_conninfo = host=192.168.11.19 user=stream_barman dbname=database_name password=password
backup_method = postgres
retention_policy_mode = auto
streaming_archiver = on
slot_name = barman
```

接著修改 /etc/barman.conf

```
[barman]
barman_user = barman
configuration_files_directory = /etc/barman.d
barman_home = /var/lib/barman
log_file = /var/log/barman/barman.log
log_level = DEBUG
compression = gzip
immediate_checkpoint = true
basebackup_retry_times = 3
basebackup_retry_sleep = 30
last_backup_maximum_age = 1 DAYS
```

基本上這樣就設定完成了

##### 檢查設定

barman 有一些指令可以用來檢查目前的設定

barman show-server hqs019 可以看到所有的設定，這裡的 hqs019 跟 barman.d/hqs019.conf 裡面用"[  ]" 包起來的名稱要一致
```
barman@barman:~$ barman show-server hqs019
Server hqs019:
	active: True
	archiver: False
	archiver_batch_size: 0
	backup_directory: /var/lib/barman/hqs019
	backup_method: postgres
	backup_options: BackupOptions(['concurrent_backup'])
	bandwidth_limit: None
	barman_home: /var/lib/barman
	barman_lock_directory: /var/lib/barman
	basebackup_retry_sleep: 30
	basebackup_retry_times: 3
	basebackups_directory: /var/lib/barman/hqs019/base
	check_timeout: 30
	compression: gzip
	config_file: /etc/postgresql/10/main/postgresql.conf
	connection_error: None
	conninfo: host=192.168.11.19 user=barman dbname=database_name password=barmanpassword
	current_size: 142740768562
	current_xlog: 0000000100000264000000BA
	custom_compression_filter: None
	custom_decompression_filter: None
	data_directory: /database
	description: hqs019
	disabled: False
	errors_directory: /var/lib/barman/hqs019/errors
	hba_file: /etc/postgresql/10/main/pg_hba.conf
	ident_file: /etc/postgresql/10/main/pg_ident.conf
	immediate_checkpoint: True
	incoming_wals_directory: /var/lib/barman/hqs019/incoming
	is_in_recovery: False
	is_superuser: True
	last_backup_maximum_age: 1 day (WARNING! latest backup is No available backups old)
	max_incoming_wals_queue: None
	minimum_redundancy: 0
	msg_list: []
	name: hqs019
	network_compression: False
	parallel_jobs: 1
	path_prefix: None
	pg_basebackup_bwlimit: True
	pg_basebackup_compatible: True
	pg_basebackup_installed: True
	pg_basebackup_path: /usr/bin/pg_basebackup
	pg_basebackup_tbls_mapping: True
	pg_basebackup_version: 10.10-0ubuntu0.18.04.1)
	pg_receivexlog_compatible: True
	pg_receivexlog_installed: True
	pg_receivexlog_path: /usr/bin/pg_receivewal
	pg_receivexlog_supports_slots: True
	pg_receivexlog_synchronous: False
	pg_receivexlog_version: 10.10-0ubuntu0.18.04.1)
	pgespresso_installed: False
	post_archive_retry_script: None
	post_archive_script: None
	post_backup_retry_script: None
	post_backup_script: None
	pre_archive_retry_script: None
	pre_archive_script: None
	pre_backup_retry_script: None
	pre_backup_script: None
	recovery_options: RecoveryOptions([])
	replication_slot: Record(slot_name='barman', active=True, restart_lsn='264/BA000000')
	replication_slot_support: True
	retention_policy: None
	retention_policy_mode: auto
	reuse_backup: None
	server_txt_version: 10.10
	slot_name: barman
	ssh_command: None
	streaming: True
	streaming_archiver: True
	streaming_archiver_batch_size: 0
	streaming_archiver_name: barman_receive_wal
	streaming_backup_name: barman_streaming_backup
	streaming_conninfo: host=192.168.11.19 user=stream_barman dbname=database_name password=password
	streaming_supported: True
	streaming_wals_directory: /var/lib/barman/hqs019/streaming
	synchronous_standby_names: ['']
	systemid: 6688476041000599317
	tablespace_bandwidth_limit: None
	timeline: 1
	wal_level: replica
	wal_retention_policy: main
	wals_directory: /var/lib/barman/hqs019/wals
	xlogpos: 264/BA000F08
```

然後用 barman check hqs019 來檢查config 有沒有問題

```
barman@barman:~$ barman check hqs019
Server hqs019:
	PostgreSQL: OK
	is_superuser: OK
	PostgreSQL streaming: OK
	wal_level: OK
	replication slot: OK
	directories: OK
	retention policy settings: OK
	backup maximum age: FAILED (interval provided: 1 day, latest backup age: No available backups)
	compression settings: OK
	failed backups: OK (there are 0 failed backups)
	minimum redundancy requirements: OK (have 0 backups, expected at least 0)
	pg_basebackup: OK
	pg_basebackup compatible: OK
	pg_basebackup supports tablespaces mapping: OK
	pg_receivexlog: OK
	pg_receivexlog compatible: OK
	receive-wal running: OK
	archiver errors: OK
barman@barman:~$
```
那個backup maximum age FAILED 不用管他，因為都還沒跑過備份，這邊錯誤是正常的

其他都OK 的話，就可以開始備份了

barman backup hqs019

```
barman@ubuntu:~$ barman backup hqs019
Starting backup using postgres method for server hqs019 in /var/lib/barman/hqs019/base/20190823T082258
Backup start at LSN: 264/A10001A8 (0000000100000264000000A1, 000001A8)
Starting backup copy via pg_basebackup for 20190823T082258
WARNING: pg_basebackup does not copy the PostgreSQL configuration files that reside outside PGDATA. Please manually backup the following files:
	/etc/postgresql/10/main/postgresql.conf
	/etc/postgresql/10/main/pg_hba.conf
	/etc/postgresql/10/main/pg_ident.conf

Copy done (time: 1 hour, 6 minutes, 28 seconds)
Finalising the backup.
Backup size: 133.0 GiB
Backup end at LSN: 264/A3000060 (0000000100000264000000A3, 00000060)
Backup completed (start time: 2019-08-23 08:22:58.116372, elapsed time: 1 hour, 6 minutes, 28 seconds)
Processing xlog segments from streaming for hqs019
	0000000100000264000000A2

barman@ubuntu:~$ 
```

跑完可以用 barman list-backup hqs019 檢查
```
barman@ubuntu:~$ barman list-backup hqs019
hqs019 20190823T082258 - Thu Aug 22 17:29:26 2019 - Size: 133.0 GiB - WAL Size: 0 B (tablespaces: tablespace_a:/var/lib/postgresql/10/main/tablespace_A, tablespace_b:/var/lib/postgresql/10/main/tablespace_B)
```

要刪除的話，要加入 backupID

```
barman@ubuntu:~$ barman delete hqs019 20190822T171355
Deleting backup 20190822T171355 for server hqs019
Delete associated WAL segments:
	00000001000002640000009F
	0000000100000264000000A0
	0000000100000264000000A1
Deleted backup 20190822T171355 (start time: Fri Aug 23 09:36:43 2019, elapsed time: 3 seconds)

```

restore 的部份，暫時沒有測試，我想應該是要找時間測試看看怎麼還原才對

不過呢，前面有提到，用streaming backup ，每一次備份都是完整備份，非常的佔用空間、時間、頻寬

所以還是要來試試看用rsync/SSH 備份的機制


