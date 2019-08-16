---
title: "[筆記] 在edgerouter上用wireguard 建立site to site VPN / Site to Site Vpn Using Wireguard in Two Edgerouters"
date: 2019-08-06T17:14:17+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-5.jpg"
categories: ['筆記']
tags: ['vpn','edgerouter']
author: "Eric Chang"
---

之前總部和分公司之間 是用buffalo 的小AP 灌 openwrt 

然後用strongswan 來打 IPSEC site to site VPN

config 看起來不是很難 (只是看起來)

但是實際上已經找不到當初的文件

所以要維護很困難(光那些RSA KEY 就不知道為何、如何產生)

後來採購了兩台edgerouter X 做測試

也用openvpn 成功的建立了 site to site VPN

本來想說 openvpn 已經夠簡單了

今天看到文章說用wireguard 可以更簡單

於是研究了一下，發現還真的很簡單！

<!--more-->
### download deb for your edgerouter
#### go check https://github.com/Lochnair/vyatta-wireguard first

```
curl -L -O https://github.com/Lochnair/vyatta-wireguard/releases/download/0.0.20190702-1/wireguard-v2.0-e50-0.0.20190702-1.deb
dpkg -i wireguard-v2.0-e50-0.0.20190702-1.deb
```

process log

```
root@ubnt112:~# dpkg -i wireguard-v2.0-e50-0.0.20190702-1.deb 
Selecting previously unselected package wireguard.
(Reading database ... 37024 files and directories currently installed.)
Preparing to unpack wireguard-v2.0-e50-0.0.20190702-1.deb ...
Adding 'diversion of /opt/vyatta/share/perl5/Vyatta/Interface.pm to /opt/vyatta/share/perl5/Vyatta/Interface.pm.vyatta by wireguard'
Adding 'diversion of /opt/vyatta/share/vyatta-cfg/templates/firewall/options/mss-clamp/interface-type/node.def to /opt/vyatta/share/vyatta-cfg/templates/firewall/options/mss-clamp/interface-type/node.def.vyatta by wireguard'
Unpacking wireguard (0.0.20190702-1) ...
Setting up wireguard (0.0.20190702-1) ...
```

#### generate private/public key in left router

```
wg genkey | tee /dev/tty | wg pubkey
```

first one in private key and the next one is public key of this router

```
QGAUHJSDFAdkfjskdjo1DP8H1NuLTrXH6kue6kphaQk/iAkc=
ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU=
```

#### configure left site edgerouter 

```
configure
set interfaces wireguard wg0 address 192.168.99.1/24
set interfaces wireguard wg0 listen-port 51820
set interfaces wireguard wg0 route-allowed-ips true
### paster your private key which was just been generate 
set interfaces wireguard wg0 private-key QGAUHJSDFAdkfjskdjo1DP8H1NuLTrXH6kue6kphaQk/iAkc=
```

#### generate private/public key in right router
```
wg genkey | tee /dev/tty | wg pubkey
```

first one in private key and the next one is public key of this router

```
UBzmPabcdefghijklmnopqrlbi5tnsQqjoJ4+H4=
tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk=
```

#### configure right site edgerouter
```
configure
set interfaces wireguard wg0 address 192.168.99.2/24 
set interfaces wireguard wg0 listen-port 51820 
set interfaces wireguard wg0 route-allowed-ips true
### paster your private key which was just been generate
set interfaces wireguard wg0 private-key UBzmPabcdefghijklmnopqrlbi5tnsQqjoJ4+H4=
```

now , configure both router to talk to each other

#### configure in left router
```
### use the right router public key here
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= allowed-ips 192.168.99.0/16
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= endpoint 222.222.222.222:51820
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= persistent-keepalive 15
```

#### configre in right router
```
### use the left router public key here
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= allowed-ips 192.168.99.0/16
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= endpoint 111.111.111.111:51280
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= persistent-keepalive 15
```

#### configure firewall policy in left site router
```
### change 40 to your own rule number
set firewall name WAN_LOCAL rule 40 source port 51820
set firewall name WAN_LOCAL rule 40 destination port 51820
```

#### configure firewall policy in right site router
```
### change 40 to your own rule number
set firewall name WAN_LOCAL rule 40 source port 51820
set firewall name WAN_LOCAL rule 40 destination port 51820
```

then finally , commit these changes on both side router
```
commit
### and save if you want
save
```

#### oops , one more step , add static route
##### manually add static route in left router
```
ip route add 192.168.111.0/24 dev wg0
```

##### manually add static route in right router
```
ip route add 192.168.112.0/24 dev wg0
```

#### check wireguard status in both router
##### left
```
 root@ubnt112:~# sudo wg
interface: wg0
  public key: ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU=
  private key: (hidden)
  listening port: 51820

peer: tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk=
  endpoint: 111.111.111.111:51820
  allowed ips: 192.168.99.0/16
  latest handshake: 1 minute, 19 seconds ago
  transfer: 7.49 MiB received, 195.86 MiB sent
  persistent keepalive: every 15 seconds
root@ubnt112:~#
```

##### right
```
interface: wg0
  public key: tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk=
  private key: (hidden)
  listening port: 51820

peer: ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU=
  endpoint: 222.222.222.222:51820
  allowed ips: 192.168.99.0/16
  latest handshake: 1 minute, 48 seconds ago
  transfer: 195.60 MiB received, 8.07 MiB sent
  persistent keepalive: every 15 seconds
root@ubnt111:~#
```

### need more edgerouter and lease line to try multiple site to site VPN using wideguard

##### need to study about allowed-ips

### sort out scripts 
##### left router
```
wg genkey | tee /dev/tty | wg pubkey
QGAUHJSDFAdkfjskdjo1DP8H1NuLTrXH6kue6kphaQk/iAkc=
ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU=
configure
set interfaces wireguard wg0 address 192.168.99.1/24
set interfaces wireguard wg0 listen-port 51820
set interfaces wireguard wg0 route-allowed-ips true
set interfaces wireguard wg0 private-key QGAUHJSDFAdkfjskdjo1DP8H1NuLTrXH6kue6kphaQk/iAkc=
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= allowed-ips 192.168.99.0/16
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= endpoint 222.222.222.222:51820
set interfaces wireguard wg0 peer tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk= persistent-keepalive 15
set firewall name WAN_LOCAL rule 40 action accept
set firewall name WAN_LOCAL rule 40 protocol udp
set firewall name WAN_LOCAL rule 40 source port 51820
set firewall name WAN_LOCAL rule 40 destination port 51820
commit
save
ip route add 192.168.111.0/24 dev wg0
```
##### right router
```
wg genkey | tee /dev/tty | wg pubkey
UBzmPabcdefghijklmnopqrlbi5tnsQqjoJ4+H4=
tmlrPSabcdefghijklmnopqrIb1Enzf+108yotkhdRmk=
configure
set interfaces wireguard wg0 address 192.168.99.2/24 
set interfaces wireguard wg0 listen-port 51820 
set interfaces wireguard wg0 route-allowed-ips true
set interfaces wireguard wg0 private-key UBzmPabcdefghijklmnopqrlbi5tnsQqjoJ4+H4=
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= allowed-ips 192.168.99.0/16
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= endpoint 111.111.111.111:51280
set interfaces wireguard wg0 peer ta+GJCWNUHJSDFAdkfjskdjnkppY5FpsIs3a8dc4oArtV8FU= persistent-keepalive 15
set firewall name WAN_LOCAL rule 40 action accept
set firewall name WAN_LOCAL rule 40 protocol udp
set firewall name WAN_LOCAL rule 40 source port 51820
set firewall name WAN_LOCAL rule 40 destination port 51820
commit
save
ip route add 192.168.112.0/24 dev wg0
```


