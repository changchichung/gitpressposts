---
title: "[筆記] 在 ubuntu 20.04 上安裝 wireguard + pihole 作 AD Blocking/install wireguard and pihole to do ad block in ubuntu 20.04 "
date: 2020-08-13T14:22:05+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-16.jpg
tags: ['wireguard','pihole','ubuntu']
author: "Eric Chang"
keywords:
  - wireguard
  - Pihole
---

手機上的廣告越來越討厭了

但是用手機看頁面、影片的機會越來越高

所以一直想看看有沒有什麼方式可以解決這個問題

不只可以用在safari 上，連APP 裡面的廣告最好都能夠擋掉

在github上面看到有個專案是 wireguard + pihole

滿有趣的，就來研究一下

<!--more-->

我在google cloud console 申請了一台free tier 的 google compute engine (真難念，就叫VPS吧)

免費的Google VPS 只能選擇美洲地區的機房，有點可惜，多少還是會有點影響

作業系統選 ubuntu 20.04 minimal

然後因為我習慣wireguard 的port 都設定在 12000

所以要記得去開啟firewall 的 UDP 12000，然後套用在這台VPS上

還有要設定 ssh 金鑰 這些都算是google compute engine 的基本設定，就不多說了

系統的基本安裝完成後，接下來要用人家寫好的 script 來安裝 wireguard + pihole

---

#### 安裝基本套件

因為是選擇 ubuntu 20.04 minimal 所以有很多套件都沒有，要先安裝這些基本套件

```
sudo apt update && sudo apt install -y vim git net-tools software-properties-common iptables python3-pip qrencode
```

#### 取得安裝script

```
mkdir git && cd git
git clone https://github.com/racbart/wireguard-pihole
```

#### 修改 install.sh

因為我的目的是只想要把DNS 查詢透過wireguard 丟去 pihole 

而不是把所有流量都轉給wireguard

所以要修改一下剛剛clone 下來的 script

```
cd wireguard-pihole
vim install.sh
```

有點忘了改了哪些東西，就大概說一下吧

##### IPV4_ADDRESS

原本的判斷VPS WAN IP 的指令在GCE上會抓到private ip

所以要改一下，在 install.sh 中找到底下這行註解掉，並修改成其他指令

```
#IPV4_ADDRESS=$(ip addr list "$INTERFACE" | grep "inet " | xargs | cut -d " " -f 2)
IPV4_ADDRESS=$(dig +short myip.opendns.com @resolver1.opendns.com)
```

##### install wireguard in ubuntu 20.04

ubuntu 20.04 安裝wireguard 的方式和 18.04 有點差別，需要多裝一個 wireguard-dkms

找到底下這行，註解掉，改成我們要的指令，python-pip 我們用 python3-pip 取代

在一開始就已經先裝了，所以這邊不需要再裝一次

```
#apt install -y wireguard python-pip
apt install -y wireguard wireguard-dkms
```

##### 啟用query logging

找到底下這行，註解掉，改成啟用 query logging

```
#QUERY_LOGGING=false
QUERY_LOGGING=true
```

存檔後離開

然後執行

```
sudo ./install.sh
```

開始進行安裝，基本上是全自動的，應該沒有錯誤，可以順利跑完 (應該啦...)

---

接著來依照我自己的需求來修改一下 add-client.sh 


##### 修改 IPV4_ADDRESS
找到底下這行註解掉，並修改成其他指令

```
#IPV4_ADDRESS=$(ip addr list "$INTERFACE" | grep "inet " | xargs | cut -d " " -f 2)
IPV4_ADDRESS=$(dig +short myip.opendns.com @resolver1.opendns.com)
```

##### 改一下 port

```
#SERVER_PORT=$(cat /etc/wireguard/wg0.conf | grep ListenPort | rev | cut -d " " -f 1 | rev)
SERVER_PORT=12000
```

##### display client and save conf 

找到底下這一段 

```
echo "
[Interface]
PrivateKey = ${CLIENT_PRIVKEY}
Address = ${NEXT_IP}/32
DNS = 10.10.0.1
[Peer]
PublicKey = ${SERVER_PUBKEY}
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = ${SERVER_ADDRESS}:${SERVER_PORT}
"
```

改成

```
# Display client config

echo "
[Interface]
PrivateKey = ${CLIENT_PRIVKEY}
Address = ${NEXT_IP}/32
DNS = 10.10.0.1

[Peer]
PublicKey = ${SERVER_PUBKEY}
#forware dns queries only,if want to forward all traffic , replace 10.10.0.1/32 to 0.0.0.0/0
AllowedIPs = 10.10.0.1/32 
Endpoint = ${SERVER_ADDRESS}:${SERVER_PORT}"|tee ${CLIENT_NAME}.conf && qrencode -t ansiutf8 -l L < ${CLIENT_NAME}.conf
```

之後要新增 client

就只要輸入
```
sudo bash add-client.sh "CLIENT_NAME"
```

就會在當前目錄底下產生 ${CLIENT_NAME}.conf 的設定檔，並顯示 qrcode

而且也不用去管 client ip 發到哪了，script 會自己去計算

再次強調，這只會把手機上的 dns 查詢透過wireguard指向到 pihole

並不會把整個流量都改從wireguard 出去

如果要改成都走wireguard 出去，那就把最後一段的 Endpoint 後面改成 0.0.0.0/0 

PC的話，wireguard 連上之後，要去手動修改DNS 

成功的話，在PC上可以看到這樣的查詢結果

```
peer: mVRp+fjHKW1/n/j5Cwn9zOlLsgtHsvoiNHPSn4bHLHg=
  endpoint: 23.34.45.67:12000
  allowed ips: 10.10.0.1/32
  latest handshake: 1 hour, 48 minutes, 39 seconds ago
  transfer: 1.46 KiB received, 1.30 KiB sent

2020-08-13 15:42:11 [root@hqdc039 wireguard]$ dig @10.10.0.1 www.google.com.tw

; <<>> DiG 9.16.1-Ubuntu <<>> @10.10.0.1 www.google.com.tw
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 25532
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.google.com.tw.		IN	A

;; ANSWER SECTION:
www.google.com.tw.	297	IN	A	64.233.177.94

;; Query time: 399 msec
;; SERVER: 10.10.0.1#53(10.10.0.1)
;; WHEN: Thu Aug 13 15:42:24 CST 2020
;; MSG SIZE  rcvd: 79

2020-08-13 15:42:24 [root@hqdc039 wireguard]$ 
```

### UPDATE

更新一下開啟 pihole DOT (DNS Over TLS) 的方式

##### 安裝並建立相關目錄

```bash
sudo apt update
sudo apt install stubby
sudo mkdir /var/cache/stubby
```

##### 修改 /etc/stubby/stubby.yml

編輯 /etc/stubby/stubby.yml 改成以下內容

```yaml
resolution_type: GETDNS_RESOLUTION_STUB

dns_transport_list:
  - GETDNS_TRANSPORT_TLS

tls_authentication: GETDNS_AUTHENTICATION_REQUIRED

tls_query_padding_blocksize: 128

edns_client_subnet_private : 1

round_robin_upstreams: 1

idle_timeout: 10000

tls_connection_retries: 5

xtls_ca_path: "/etc/ssl/certs/"

################################ LISTEN ADDRESS ################################
# Set the listen addresses for the stubby DAEMON. This specifies localhost IPv4
# and IPv6. It will listen on port 53 by default. Use <IP_address>@<port> to
# specify a different port
listen_addresses:
  - 127.0.0.1@5453

appdata_dir: "/var/cache/stubby"

upstream_recursive_servers:
   - address_data: 145.100.185.15
     tls_auth_name: "dnsovertls.sinodun.com"
   - address_data: 1.1.1.1
     tls_auth_name: "cloudflare-dns.com"

## Quad 9 'secure' service - Filters, does DNSSEC, doesn't send ECS
#  - address_data: 9.9.9.9
#    tls_auth_name: "dns.quad9.net"
## Quad 9 'insecure' service - No filtering, does DNSSEC, may send ECS (it is
## unclear if it honours the edns_client_subnet_private request from stubby)
#  - address_data: 9.9.9.10
#    tls_auth_name: "dns.quad9.net"
## Cloudflare 1.1.1.1 and 1.0.0.1
#  - address_data: 1.1.1.1
#    tls_auth_name: "cloudflare-dns.com"
#  - address_data: 1.0.0.1
#    tls_auth_name: "cloudflare-dns.com"
## The Uncensored DNS servers
#  - address_data: 89.233.43.71
#    tls_auth_name: "unicast.censurfridns.dk"
#    tls_pubkey_pinset:
#      - digest: "sha256"
#        value: wikE3jYAA6jQmXYTr/rbHeEPmC78dQwZbQp6WdrseEs=
## Fondation RESTENA (NREN for Luxembourg)
#  - address_data: 158.64.1.29
#    tls_auth_name: "kaitain.restena.lu"
#    tls_pubkey_pinset:
#      - digest: "sha256"
#        value: 7ftvIkA+UeN/ktVkovd/7rPZ6mbkhVI7/8HnFJIiLa4=
## Google
#  - address_data: 8.8.8.8
#    tls_auth_name: "dns.google"
#  - address_data: 8.8.4.4
#    tls_auth_name: "dns.google"

```

重啟 stubby service

```shell
sudo service tubby restart
```

##### 修改pihole 相關設定

開啟 pihole web 管理界面 settings -> dns -> 左邊預設的DNS 都不要選，在右邊的 custom 1(IPV4) 填入 

存檔後離開
```
127.0.0.1#5453
```


