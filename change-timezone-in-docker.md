---
title: "[筆記] 修改 docker 容器內的時區 - Change Timezone in Docker"
date: 2019-05-21T17:25:15+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-3.jpg"
categories: ['筆記']
tags: ['docker','timezone']
author: "Eric Chang"
---

最近一直在玩一些docker，不過老是會碰到歪果扔寫的東西，時區都不一致

有的用 UTC，有的用localtime，就是沒碰到用 Asia/Taipei 的....

<!--more-->
之前因為沒有要上線，所以這個問題可以當烏龜忽略過去

不過測試了一陣子的librenms 打算正式上線了

又不想重作一次，導致累積滿久的圖表資料都消失

所以開始找尋方法來面對這個問題

本來看了這篇 https://www.arthurtoday.com/2016/07/how-to-setup-docker-container-timezone-host.html

想說來試試看好了

不過呢，一開始依照這篇的說明，在docker-compose.yml 內加入
```
web:
  image: jarischaefer/docker-librenms
  hostname: librenms
  ports:
    - "8001:80"
    - "44301:443"
  volumes:
    - /etc/hosts:/etc/hosts
    - /etc/localtime:/etc/localtime:ro
  environment:
    - TZ="Asia/Taipei"
```

重起之後沒作用 0rz

於是我又加入了

```
  volumes:
    - /etc/hosts:/etc/hosts
    - /etc/localtime:/etc/localtime:ro
    - /etc/timezone:/etc/timezone:ro
```

結果啟動直接報錯誤了..

看一下 log

```
May 21 09:09:12 bbs012 syslog-ng[12]: syslog-ng starting up; version='3.13.2'
*** Running /etc/my_init.d/librenms_100_cron...
*** Running /etc/my_init.d/librenms_101_ssl...
*** Running /etc/my_init.d/librenms_102_ipv6...
*** Running /etc/my_init.d/librenms_103_timezone...
/etc/my_init.d/librenms_103_timezone: line 4: /etc/timezone: Read-only file system
*** /etc/my_init.d/librenms_103_timezone failed with status 1

*** Killing all processes...
```

所以這個檔案不能改成 ro ，不過如果不能唯獨，那啟動docker的時候應該會被蓋掉吧？

先來看一下那個 librenms_103_timezone 在幹什麼好了

```
docker exec -it librenms_web_1 cat /etc/my_init.d/librenms_103_timezone
#!/bin/bash -e

if [ -n "$TZ" ]; then
	ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

	if [ ! -f /etc/php/7.3/cli/conf.d/100-timezone.ini ]; then
		echo "date.timezone=$TZ" > /etc/php/7.3/cli/conf.d/100-timezone.ini
	fi

	if [ ! -f /etc/php/7.3/fpm/conf.d/100-timezone.ini ]; then
		echo "date.timezone=$TZ" > /etc/php/7.3/fpm/conf.d/100-timezone.ini
	fi
fi
2019-05-21 17:33:46 [mini@s013 librenms]$
```

OK ，這裡的確是用傳進來的 $TZ 在設定timezone 沒錯

可是我前面已經改過 docker-compose.yml 

有把 TZ帶進來了啊？再來確認一下

```
docker exec -it librenms_web_1 cat /etc/timezone
"Asia/Taipei"

```

咦，沒錯啊？ 欸斗，等等，那兩個 "" 有點刺眼...

跟本機的比對一下看看

```
2019-05-21 17:36:20 [mini@s013 librenms]$ docker exec -it librenms_web_1 cat /etc/timezone
"Asia/Taipei"
2019-05-21 17:36:23 [mini@s013 librenms]$ cat /etc/timezone
Asia/Taipei
2019-05-21 17:37:10 [mini@s013 librenms]$ 
```

嗯，的確，本機的格式的確不包含那兩個 ""

那就改掉再來試試看吧...改成底下這樣

```
  environment:
    - TZ=Asia/Taipei

```

重起 docker ，然後確認時間看看

```
2019-05-21 17:39:00 [mini@s013 librenms]$ docker-compose down;docker-compose up -d
Stopping librenms_web_1   ... done
Stopping librenms_mysql_1 ... done
Removing librenms_web_1   ... done
Removing librenms_mysql_1 ... done
Removing network librenms_default
Creating network "librenms_default" with the default driver
Creating librenms_mysql_1 ... done
Creating librenms_web_1   ... done
2019-05-21 17:39:22 [mini@s013 librenms]$ docker exec -it librenms_web_1 cat /etc/timezone
Asia/Taipei
2019-05-21 17:39:42 [mini@s013 librenms]$ docker exec -it librenms_web_1 date
Tue May 21 17:39:48 CST 2019
2019-05-21 17:39:48 [mini@s013 librenms]$ 
```

OK ，果然沒有問題了！

雖然是小小的"" ，還是要特別注意啊！

