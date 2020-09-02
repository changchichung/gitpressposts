---
title: "[筆記] 在 ubuntu 20.04 底下，用certbot 透過Cloudflare 申請全域的 Letsencrypt 憑證"
date: 2020-09-02T15:55:40+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-4.jpg
tags: ['certbot','Cloudflare','Letsencrypt']
author: "Eric Chang"
keywords:
  - certbot
  - Cloudflare
  - Letsencrypt
---

之前用caddy 作為反向代理，其中一個優勢就是caddy 會自動處理Letsencrypt 憑證的問題

也不用煩惱怎麼去更新一堆有的沒的

不過，實際應用上，還是偶爾會拿這些憑證檔案來用的狀況

雖然可以從caddy 上面取得這些檔案

但是基本上這些檔案都是綁定一個特定的hostname

可是我想要有一個憑證，可以給同網域底下的機器用 ( Wildcard certificates )

<!--more-->

要申請Wildcard certificates ，必須要採用 DNS 驗證的方式

一般手動操作的步驟，會先產生一組亂數字串，然後更新 DNS 上面去

如果要改成自動化，要多一些步驟

### 安裝 certbot 及 Cloudflare 外掛

首先，先來安裝會用到的套件

```
sudo apt install certbot letsencrypt python3-certbot-dns-cloudflare
```

### 設定 cloudflare API

這個步驟我測了好久，網路上的說明似乎都過期了，造成cloudflare API 那邊會發生錯誤

先登入 cloudflare 管理界面的API token 設定

https://dash.cloudflare.com/profile/api-tokens

建立一組token

內容如下

!['cloudflare API']('https://i.imgur.com/3dZN6qC.png')

在權限設定的地方，選擇三個項目

zone-zone settings-edit
zone-zone-edit
zone-DNS-edit

在下一個 zone resources 選擇 include-All zones

存檔後會產生一組 API token ，接著就是用這組 token 來做DNS更新

### 編輯 cloudflare 設定檔

在 /etc底下新增一個 cloudflare.ini

內容如下

```
sudo vim /etc/cloudflare.ini

dns_cloudflare_email =  #email@address.here
dns_cloudflare_api_key = #API token here
```

存檔後離開，然後改一下權限，不然等一下certbot 會跳警告

```
sudo chmod 0600 /etc/cloudflare.ini
```

### 執行certbot 取得憑證

執行以下的指令
```
sudo certbot certonly  --dns-cloudflare --dns-cloudflare-credentials /etc/cloudflare.ini --preferred-challenges=dns --email admin@abc.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d abc.com -d *.abc.com
```



正常的話，會是這樣的結果

```
sudo certbot certonly  --dns-cloudflare --dns-cloudflare-credentials /etc/cloudflare.ini --preferred-challenges=dns --email admin@abc.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d abc.com -d *.abc.com

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator dns-cloudflare, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for abc.com
dns-01 challenge for abc.com
Waiting 10 seconds for DNS changes to propagate
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/abc.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/abc.com/privkey.pem
   Your cert will expire on 2020-12-01. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```
 
這樣子就取得了全域通用的SSL 憑證檔案

如果看到底下這種錯誤

```
administrator@ubuntu:~$ sudo certbot certonly  --dns-cloudflare --dns-cloudflare-credentials /etc/cloudflare.ini --preferred-challenges=dns --email admin@abc.com --server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d abc.com -d *.abc.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator dns-cloudflare, Installer None
Obtaining a new certificate
Performing the following challenges:
dns-01 challenge for abc.com
dns-01 challenge for abc.com
Cleaning up challenges
Error determining zone_id: 6003 Invalid request headers. Please confirm that you have supplied valid Cloudflare API credentials. (Did you copy your entire API key?)
```

那就是cloudflare API 那邊的權限設定錯了，我就是在這邊卡很久...

請參照上面的步驟和圖片正確的設定

可以用 certbot certificates 來驗證看看

```
administrator@ubuntu:~$ sudo certbot certificates
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: abc.com
    Domains: abc.com *.abc.com
    Expiry Date: 2020-12-01 05:31:31+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/abc.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/abc.com/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

之後就可以用

```
sudo certbot renew
```

來更新憑證

寫到/etc/crontab 去排程每個月的1號自動更新

```
administrator@ubuntu:~$ echo "* * 1 * * root /usr/bin/certbot renew" |sudo tee -a /etc/crontab
* * 1 * * root /usr/bin/certbot renew
administrator@ubuntu:~$ 
```

接下來就等三個月之後，檢查看看憑證是否有自動更新了！


