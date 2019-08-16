---
title: "[筆記] 紀錄所有下過的指令、時間 / Log All commands with timestamp"
date: 2019-04-23T15:08:36+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['log']
author: "Eric Chang"
---
今天發生一件有點詭異的事情，本來應該要經過某個指令才會產生的檔案

居然不知為何自己產生了，在我記憶中沒有去執行過那個指令

翻了一下 bash_history ，裡面也只有下過哪些指令，沒有紀錄時間，完全沒有參考價值(攤手)

所以翻了一下網路，至少把這兩台主要跑ansible的機器的log功能補上紀錄所有指令以及時間的部份

<!--more-->

參考這個網頁
![https://askubuntu.com/questions/93566/how-to-log-all-bash-commands-by-all-users-on-a-server](https://askubuntu.com/questions/93566/how-to-log-all-bash-commands-by-all-users-on-a-server)

我沒有打算要紀錄「所有」使用者的指令，只要看有權力執行重要指令的帳號就好

所以先用minion(管理用的帳戶)登入後

先編輯 ~/.bashrc
加入
```
export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" ) [$RETRN_VAL]"'
```

因為這邊用到syslog 的 local6，所以要跟著修改 syslog的設定

```
sudo vim /etc/rsyslog.d/bash.conf

加入這行

local6.*    /var/log/commands.log

接著設定讓/var/log/commands.log 也能夠自動輪替

sudo vim /etc/logrotate.d/rsyslog

在適當的位置 加入 /var/log/commands.log

然後重起 rsyslog

sudo service rsyslog restart
```

用 minion 登出登入後，就可以看到所有指令都被完整的紀錄下來了
```
sudo cat /var/log/commands.log

2019-04-23 15:18:48 [minion@hqs010 ~]$ sudo cat /var/log/commands.log 
Apr 23 15:06:51 hqs010 minion: minion [30832]:  [0]
Apr 23 15:06:53 hqs010 minion: minion [30832]: ls -lart [0]
Apr 23 15:06:55 hqs010 minion: minion [30832]: ls -alrt /tmp/ [0]
Apr 23 15:06:58 hqs010 minion: minion [30832]: ls -lart /var/log/ [0]
Apr 23 15:07:07 hqs010 minion: minion [30832]: sudo cat  /var/log/commands.log  [0]
Apr 23 15:07:13 hqs010 minion: minion [30832]: ls -lart /tmp/ [0]
Apr 23 15:07:18 hqs010 minion: minion [30832]: cat /tmp/hqs010_inventory.txt  [0]
Apr 23 15:07:22 hqs010 minion: minion [30832]: cd [0]
Apr 23 15:07:22 hqs010 minion: minion [30832]: ls [0]
Apr 23 15:07:24 hqs010 minion: minion [30832]: ls -lart [0]
Apr 23 15:07:28 hqs010 minion: minion [30832]: ls .inxi/ [0]
Apr 23 15:07:35 hqs010 minion: minion [30832]: clear [0]
Apr 23 15:18:48 hqs010 minion: minion [30832]: ip addr [0]
2019-04-23 15:18:55 [minion@hqs010 ~]$ 
```
裡面應該會看到滿滿的 cd / ls / cat 吧  XD

