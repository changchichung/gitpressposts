---
title: "[筆記] 用pg_upgrade指令升級postgresql 15 -> 16 beta1"
date: 2023-06-29T16:08:15+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://raw.githubusercontent.com/changchichung/imagebed/master/post-default-05.jpg
tags: ['postgresql']
author: "Eric Chang"
keywords:
  - postgresql
---

老闆在slack 上說，要試試看透過 pg_upgrade 的指令來升級現有的資料庫

以前都是dump DB , remove packages , install new packages , restore DB

我也不曉得為什麼這次突然又想要用 pg_upgrade

反正人家有交待，那我們就抱著學習的心態來試試看

<!--more-->

整個過程的指令整理如下，因為中間需要轉換幾次身分還有檢查指令的結果
所以就不寫成 script 了，丟去 wiki 上面讓同事可以copy & paste 就好了
最後的檢查DB輸出結果那邊可以不用執行

```
### START ###
### login server as minion

# import repository key
sudo apt install curl ca-certificates gnupg
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/apt.postgresql.org.gpg >/dev/null

# check pgdg.list
echo "deb http://apt.postgresql.org/pub/repos/apt/ jammy-pgdg-snapshot main 16" |sudo tee -a /etc/apt/sources.list.d/pgdg.list

# install psql-16
sudo apt update
sudo apt install postgresql-16 -y

# make new database folder
sudo mkdir /database_16
sudo chown -R postgres:postgres /database_16

### cp psql configurations

# stop psql
sudo service postgresql stop

### remove old systemd files and create a new one
sudo systemctl disable postgresql@15
sudo systemctl stop postgresql@15
sudo systemctl enable postgresql@16-main
sudo systemctl start postgresql@16-main

### sudo to postgres

sudo su - postgres

# update /usr/lib/postgresql/15/bin to 16
sed -i 's/15\/bin/16\/bin/g' ~/.profile
# apply .profile
source ~/.profile

#check if using new pg command now
# should return /usr/lib/postgresql/16/bin/pg_ctl
which pg_ctl

### init db
initdb -D /database_16

### cp psql configurations
cp -R /etc/postgresql/15/main/{conf.d,postgresql.conf,pg_hba.conf} /etc/postgresql/16/main/

### update postgresql configurations
sed -i 's/\/database/\/database_16/g' /etc/postgresql/16/main/postgresql.conf
sed -i 's/15\/main/16\/main/g' /etc/postgresql/16/main/postgresql.conf
sed -i 's/15-main/16-main/g' /etc/postgresql/16/main/postgresql.conf

### restart postgresql to apply configuration as minion
exit
sudo systemctl restart postgresql@16-main

### check if postgresql listen on 5432
### should return
#tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      278716/postgres
#tcp6       0      0 :::5432                 :::*                    LISTEN      278716/postgres

sudo netstat -antlp |grep 5432

### create extensions on template1 as postgres
sudo su - postgres
psql -d template1

### now u should see something like this
# psql (16beta1 (Ubuntu 16~beta1-2.pgdg22.04+~20230605.2256.g3f1aaaa))

CREATE EXTENSION if not exists pg_trgm;
CREATE EXTENSION if not exists cube;
CREATE EXTENSION if not exists earthdistance;
CREATE EXTENSION if not exists tablefunc;
CREATE EXTENSION if not exists intarray;
CREATE EXTENSION if not exists pg_stat_statements;

### back to terminal to run pg_upgrade
### now stop postgresql service first
### some error messages if u did not stop postgresql
#There seems to be a postmaster servicing the new cluster.
#Please shutdown that postmaster and try again.
#Failure, exiting
###

### stop postgresql service as minion
exit
sudo systemctl stop postgresql@16-main

### back to postgres
sudo su - postgres
### check before upgrading
pg_upgrade -d /database -D /database_16 -b /usr/lib/postgresql/15/bin -B /usr/lib/postgresql/16/bin --check

###Performing Consistency Checks
###-----------------------------
###Checking cluster versions                                   ok
###Checking database user is the install user                  ok
###Checking database connection settings                       ok
###Checking for prepared transactions                          ok
###Checking for system-defined composite types in user tables  ok
###Checking for reg* data types in user tables                 ok
###Checking for contrib/isn with bigint-passing mismatch       ok
###Checking for incompatible "aclitem" data type in user tablesok
###Checking for presence of required libraries                 ok
###Checking database user is the install user                  ok
###Checking for prepared transactions                          ok
###Checking for new cluster tablespace directories             ok

###*Clusters are compatible*
### make sure u see the messages above

### now run pg_upgrade , this will take some time
pg_upgrade -d /database -D /database_16 -b /usr/lib/postgresql/15/bin -B /usr/lib/postgresql/16/bin

### success messages
###Upgrade Complete
###----------------
###Optimizer statistics are not transferred by pg_upgrade.
###Once you start the new server, consider running:
###    /usr/lib/postgresql/16/bin/vacuumdb --all --analyze-in-stages
###Running this script will delete the old cluster's data files:
###    ./delete_old_cluster.sh

### back to minion , start postgresql service
exit
sudo systemctl start postgresql@16-main

### check if postgresql running
sudo netstat -antlp|grep 5432
### tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      279322/postgres
### tcp6       0      0 :::5432                 :::*                    LISTEN      279322/postgres

### check if some_db_here DB i sready to serve
sudo su - some_db_here
/usr/lib/postgresql/16/bin/vacuumdb --all --analyze-in-stages

psql -d some_db_here

select * from center;
### check the output

### END ###

```
