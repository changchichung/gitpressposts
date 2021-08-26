---
title: "auto fetch  Wildcard ssl certs with lego + acme-dns ( Domain Register : Namecheap)"
date: 2021-08-26T12:08:43+08:00
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-8.jpg
tags: ['acme','acme-dns','lego','ssl']
author: "Eric Chang"
keywords:
  - acme
  - lego
---

### auto fetch  Wildcard ssl certs with lego + acme-dns ( Domain Register : Namecheap)

自從用了 [leproxy](https://github.com/artyom/leproxy) 之後，其實就很少在管ssl 憑證的問題，反正[leproxy ](https://github.com/artyom/leproxy)都會自動處理好

不過LAN裡面的機器越來越多，每次看到警告說沒有加密的訊息就有點不爽，之前用了很多方式去申請全域憑證，申請倒是還好，沒太多問題。但是一碰到要更新，就都無法自動，因為都會要求去修改DNS 的 TXT 或者是 CNAME 記錄。

一般來說，如果是其他DNS 供應商，大部分都會提供API，那就還好。 BUT !! (對，然生就是離不開這個BUT ...) 我們的域名是老闆在 iwantmyname 買的，一開始是給 webfaction 代管，後來webfaction 被godaddy 買走，就轉到 namecheap 去(我也不知道為什麼不在godaddy 就好)。


<!--more-->


DNS 管理基本上都是大同小異啦，可是namecheap 免費賬戶不提供 API ，應該說要使用namecheap 提供的API ，需要滿足以下的條件

```
I want to enable API for my account. Are there any specific requirements?

We have certain requirements for activation to prevent system abuse. In order to have API enabled for your account, you should meet one of the following requirements:

- have at least 20 domains under your account;
- have at least $50 on your account balance;
- have at least $50 spent within the last 2 years.
```

之前問過老闆，可不可以丟個50 鎂在賬戶裡面，好讓我可以用API 去修改DNS 來自動取得SSL 憑證，同樣地，也不知道為什麼，連50鎂也不給存...

於是過了一段每幾個月就憑證過期，需要手動更新的日子....想想實在不甘願，本來已經想說去買一些一塊美金一年的domain 然後通通移轉到namecheap ，來滿足上面的第一個條件。但是這又要自己花錢(我已經自掏腰包很多了在這邊買LAB設備)，最後決定還是用[lego](https://github.com/go-acme/lego) + [acme-dns](https://github.com/joohoi/acme-dns) 來做

其實前兩年就有玩過 lego ，但是當時應該是功能上還沒完整，這次在找 acme-dns 的文件時，發現lego 一直有持續更新，所以這次才決定改用 lego + acme-dns 來達到「自動更新」 SSL 憑證的需求，底下就簡單說明一下設定步驟、內容

#### 取得 lego & acme-dns

lego 以及acme-dns 都是使用 golang 開發的，這也是為什麼我選用這兩個組合的原因之一，省去自己編譯還要安裝一堆有的沒的套件，兩個套件都有prebuild binary package，直接下載回來就可以了

##### lego

wget https://github.com/go-acme/lego/releases/download/v4.4.0/lego_v4.4.0_linux_amd64.tar.gz

##### acme-dns

wget https://github.com/joohoi/acme-dns/releases/download/v0.8/acme-dns_0.8_linux_amd64.tar.gz

解壓縮後取得執行檔

tar zxvf lego_v4.4.0_linux_amd64.tar.gz && sudo mv lego /usr/local/bin/
tar zxvf acme-dns_0.8_linux_amd64.tar.gz && sudo mv acme-dns /usr/local/bin/

---

##### Firewall 設定

firewall 上開啟port mapping ，把 UDP 53 轉給這臺跑 lego 的機器

如果這臺機器上有軟體已經佔用 53 port ，要想辦法先解決。

對，我說的就是那個超級討厭的 systemd-resolved

本機如果有開firewall ，記得要放行 udp 53


---

#### 設定acme-dns


```shell
#建立 acme-dns 目錄
mkdir -p /etc/acme-dns
mkdir -p /var/lib/acme-dns
#建立 acme-dns 設定檔
sudo vim /etc/acme-dns/config.cfg
```


config 的內容如下，順便補上一些自己的註解

```shell
#/etc/acme-dns/config.cfg
[general]
# DNS interface
# 本來預設是只有 :53 在某些VPS 上會出錯，所以改成 0.0.0.0:53
listen = "0.0.0.0:53"
protocol = "udp"
# domain name to serve the requests off of
# 不是要設定的 domain，而是這臺機器要負責的sub domain
# 總之就是輸入 acme 再加上原本的domain
# 不想用 acme 當然也可以
domain = "acme.abc.com"
# zone name server
# ns1 再加上原本的 domain
# 一樣，不想用ns1 也可以，後面記得作對應的修改
nsname = "ns1.abc.com"
# admin email address, where @ is substituted with .
# 管理者email , admin + 原本的 domain
nsadmin = "admin.abc.com"
# predefined records served in addition to the TXT
# 
# 前面兩筆 A 記錄對應上面的 domain , nsname
# 後面則是這臺機器的 WAN IP
# 第三筆 是NS 記錄
# 這三筆記錄等一下要新增到namecheap 的DNS 
records = [
    "acme.abc.com. A 11.22.33.44",
    "ns1.acme.abc.com. A 11.22.33.44",
    "acme.abc.com. NS ns1.abc.com.",
]
debug = false

[database]
engine = "sqlite3"
connection = "/var/lib/acme-dns/acme-dns.db"

### 要記一下port ，等等會用到
[api]
api_domain = ""
ip = "127.0.0.1"
disable_registration = false
autocert_port = "80"
port = "9000"
tls = "none"
corsorigins = [
    "*"
]
use_header = false
header_name = "X-Forwarded-For"

[logconfig]
loglevel = "debug"
logtype = "stdout"
logformat = "text"

```

編輯完後，存檔離開。



新增 acme-dns.service 的systemd config

```shell
sudo vim /etc/systemd/system/acme-dns.service
```

內容如下

```shell
# /etc/systemd/system/acme-dns.service
[Unit]
Description=ACMD DNS
After=network.target

[Service]
ExecStart=/usr/local/bin/acme-dns
Restart=on-failure

[Install]
WantedBy=multi-user.target

```

存檔離開，並啟用 acme-dns service

```shell
sudo systemctl daemon-reload 
sudo systemctl enable --now acme-dns.service
# 檢查一下狀態是否正常
sudo systemctl status acme-dns
# 底下這個指令如果沒有回傳任何訊息，是正常的
curl http://localhost:9000/health
```



#### 設定namecheap DNS 記錄

總共要新增兩筆A 記錄，一筆 NS 記錄 (目前)，後面還會需要新增一筆 CNAME

domain

![20210826113826-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210826113826-image.png)

nsname

![20210826113946-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210826113946-image.png)

NS record

![20210826114027-image.png](https://raw.githubusercontent.com/changchichung/imagebed/main/20210826114027-image.png)



然後休息個五分鐘十分鐘的，讓子彈飛一下，等DNS生效



##### 透過lego 取得憑證

只要確認上面的防火牆設定、acme-dns 設定、以及 DNS 的修改生效之後，剩下的lego 指令就很簡單了

https://go-acme.github.io/lego/dns/acme-dns/



```shell
# 第一個ACME_DNS_API_BASE是剛剛設定acme-dns API port
# 然後 ACME_DNS_STORAGE_PATH 是lego存放賬戶資料的地方
# 後面就是lego 的指令
ACME_DNS_API_BASE=http://localhost:9000 ACME_DNS_STORAGE_PATH=/home/minion/.lego-acme-dns-accounts.json lego --email changch@abc.com --dns acme-dns --domains *.abc.com run
```

執行完成後，會在目錄底下產生一個叫 .lego 的目錄，用來存放憑證檔案

```shell
2021-08-26 11:55:16 [minion@hqs058 ~]$ ls -la .lego/certificates/
total 28
drwx------ 2 minion sudo 4096 Aug 26 09:35 .
drwx------ 4 minion sudo 4096 Aug 26 09:33 ..
-rw------- 1 minion sudo 5325 Aug 26 09:35 _.abc.com.crt
-rw------- 1 minion sudo 3751 Aug 26 09:35 _.abc.com.issuer.crt
-rw------- 1 minion sudo  238 Aug 26 09:35 _.abc.com.json
-rw------- 1 minion sudo  227 Aug 26 09:35 _.abc.com.key
2021-08-26 11:58:22 [minion@hqs058 ~]$ 

```

沒錯，就這麼簡單!!

甚至於我要撤銷這些憑證也很簡單!!!

把最後面的 run 改成 revoke 就可以了！

```shell
ACME_DNS_API_BASE=http://localhost:9000 ACME_DNS_STORAGE_PATH=/home/minion/.lego-acme-dns-accounts.json lego --email changch@abc.com --dns acme-dns --domains *.abc.com revoke
2021/08/26 11:59:13 Trying to revoke certificate for domain *.abc.com
2021/08/26 11:59:14 Certificate was revoked.
2021/08/26 11:59:14 Certificate was archived for domain: *.abc.com

```

再來跑一次申請新憑證測試看看

```shell
ACME_DNS_API_BASE=http://localhost:9000 ACME_DNS_STORAGE_PATH=/home/minion/.lego-acme-dns-accounts.json lego --email changch@abc.com --dns acme-dns --domains *.abc.com run
2021/08/26 12:00:51 [INFO] [*.abc.com] acme: Obtaining bundled SAN certificate
2021/08/26 12:00:52 [INFO] [*.abc.com] AuthURL: https://acme-v02.api.letsencrypt.org/acme/authz-v3/25150773810
2021/08/26 12:00:52 [INFO] [*.abc.com] acme: authorization already valid; skipping challenge
2021/08/26 12:00:52 [INFO] [*.abc.com] acme: Validations succeeded; requesting certificates
2021/08/26 12:00:53 [INFO] [*.abc.com] Server responded with a certificate.
```

同樣地，會產生新的ssl 憑證

```shell
2021-08-26 12:00:53 [minion@hqs058 ~]$ ls -la .lego/certificates/
total 28
drwx------ 2 minion sudo 4096 Aug 26 12:00 .
drwx------ 5 minion sudo 4096 Aug 26 11:59 ..
-rw------- 1 minion sudo 5325 Aug 26 12:00 _.abc.com.crt
-rw------- 1 minion sudo 3751 Aug 26 12:00 _.abc.com.issuer.crt
-rw------- 1 minion sudo  238 Aug 26 12:00 _.abc.com.json
-rw------- 1 minion sudo  227 Aug 26 12:00 _.abc.com.key
2021-08-26 12:02:37 [minion@hqs058 ~]$ 
```

超方便的啊！！！！

後面要更新就把指令最後的 run 改成 renew

```shell
ACME_DNS_API_BASE=http://localhost:9000 ACME_DNS_STORAGE_PATH=/home/minion/.lego-acme-dns-accounts.json lego --email changch@abc.com --dns acme-dns --domains *.abc.com renew
2021/08/26 12:04:00 [*.abc.com] The certificate expires in 89 days, the number of days defined to perform the renewal is 30: no renewal.
```



因為是剛剛才要到的憑證，當然是不能更新啦...

把這個指令寫到 crontab ，以後時間到了就會自動更新憑證

後續再搭配 ansible 來抓新的憑證，派送到其他伺服器去

終於可以不用再為ssl 憑證煩惱了！！！



