---
title: "[筆記] 透過 wireguard 建立多點 site to site VPN / Multiple Site to Site VPN Using Wireguard"
date: 2019-08-13T15:50:31+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-10.jpg"
categories: ['筆記']
tags: ['vpn','ubuntu','wireguard']
author: "Eric Chang"
---

因為實在受夠了現在用的 openwrt + strongswan 建立 IPSec VPN

雖然說其實沒有什麼不好，但是畢竟不是我建立的，而當初的文件也都不見了

完全沒辦法了解當時設計的邏輯，造成後續debug 困難

可以想像一下，一台VPN router ping 不到remote、ping不到internet、甚至ping不到自己 是要怎麼debug !?(翻桌

之前買了兩台edgerouter X 拿來玩了一下 wireguard，感覺還不錯，不過只有測試到點對點

這次試試看躲在gateway後面，看看能不能建立多點的VPN環境

<!--more-->

#### every node

##### enable ip_forward 
edit /etc/sysctl.conf
add below line in the end of the file

```
net.ipv4.ip_forward=1
```

##### install wireguard

```
sudo apt-get install libmnl-dev linux-headers-$(uname -r) build-essential make git libelf-dev
git clone https://git.zx2c4.com/WireGuard
cd WireGuard/src/
make
sudo make install
```
or
**via apt**
```
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt install wireguard
```

##### create wireguard service file
add /etc/systemd/system/multi-user.target.wants/wg-quick@wg0.service
```
[Unit]
Description=WireGuard via wg-quick(8) for %I
After=network-online.target nss-lookup.target
Wants=network-online.target nss-lookup.target
Documentation=man:wg-quick(8)
Documentation=man:wg(8)
Documentation=https://www.wireguard.com/
Documentation=https://www.wireguard.com/quickstart/
Documentation=https://git.zx2c4.com/WireGuard/about/src/tools/man/wg-quick.8
Documentation=https://git.zx2c4.com/WireGuard/about/src/tools/man/wg.8

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/bin/wg-quick up %i
ExecStop=/usr/bin/wg-quick down %i
Environment=WG_ENDPOINT_RESOLUTION_RETRIES=infinity

[Install]
WantedBy=multi-user.target
```

#### Node A 

##### create wireguard private/public key
```
wg genkey > /etc/wireguard/private
cat /etc/wireguard/private | wg pubkey > /etc/wireguard/public
```
##### /etc/wireguard/wg0.conf
watch the interface name , must meets the interface name in system , ens18 is the default value of my test VM

```
[Interface]
Address = 10.0.0.40/24
ListenPort = 12000
PrivateKey = private key of node A
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE

[Peer]
PublicKey = public key of node B
AllowedIPs = 10.0.0.28/32,192.168.28.0/24
Endpoint = 2.2.2.2:12000
PersistentKeepalive = 15

[Peer]
PublicKey = public key of node C
AllowedIPs = 10.0.0.80/32,192.168.80.0/24
Endpoint = 3.3.3.3:12000
PersistentKeepalive = 15
```

#### Node B (peer 1)

##### create wireguard private/public key
```
wg genkey > /etc/wireguard/private
cat /etc/wireguard/private | wg pubkey > /etc/wireguard/public
```

##### /etc/wireguard/wg0.conf
watch the interface name , must meets the interface name in system , ens18 is the default value of my test VM

```
[Interface]
ListenPort = 12000
PrivateKey = private key of node B
Address = 10.0.0.28/24
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE

[Peer]
PublicKey = public key of node A
AllowedIPs = 10.0.0.40/32,192.168.40.0/24
Endpoint = 1.1.1.1:12000
PersistentKeepalive = 15

[Peer]
PublicKey = public key of node C
AllowedIPs = 10.0.0.80/32,192.168.80.0/24
Endpoint = 3.3.3.3:12000
PersistentKeepalive = 15

```

#### Node C (peer 2)


##### create wireguard private/public key
```
wg genkey > /etc/wireguard/private
cat /etc/wireguard/private | wg pubkey > /etc/wireguard/public
```
#### /etc/wireguard/wg0.conf

watch the interface name , must meets the interface name in system , ens18 is the default value of my test VM

```
[Interface]
ListenPort = 12000
PrivateKey = private key of node C
Address = 10.0.0.80/24
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o ens18 -j MASQUERADE


[Peer]
PublicKey = public key of node A
AllowedIPs = 10.0.0.40/32,192.168.40.0/24
Endpoint = 1.1.1.1:12000
PersistentKeepalive = 15

[Peer]
PublicKey = public key of node B
AllowedIPs = 10.0.0.28/32,192.168.28.0/24
Endpoint = 2.2.2.2:12000
PersistentKeepalive = 15
```

##### Test
Reboot all nodes , check if interface wg0 up by default or not

use command wg show to check status

for example , this is result of wg show in node C

```
root@sdvpn:~# wg show
interface: wg0
  public key: public key of Node C
  private key: (hidden)
  listening port: 12000

peer: public key of node A
  endpoint: 1.1.1.1:12000
  allowed ips: 10.0.0.40/32, 192.168.40.0/24
  latest handshake: 49 seconds ago
  transfer: 9.77 KiB received, 9.73 KiB sent
  persistent keepalive: every 15 seconds

peer: public key of node B
  endpoint: 2.2.2.2:12000
  allowed ips: 10.0.0.28/32, 192.168.28.0/24
  latest handshake: 2 minutes, 8 seconds ago
  transfer: 3.93 KiB received, 7.89 KiB sent
  persistent keepalive: every 15 seconds
```
and the ping test
```
root@sdvpn:~# ping -c 1 192.168.40.40
PING 192.168.40.40 (192.168.40.40) 56(84) bytes of data.
64 bytes from 192.168.40.40: icmp_seq=1 ttl=63 time=21.2 ms

--- 192.168.40.40 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 21.204/21.204/21.204/0.000 ms
root@sdvpn:~# ping -c 1 192.168.28.40
PING 192.168.28.40 (192.168.28.40) 56(84) bytes of data.
64 bytes from 192.168.28.40: icmp_seq=1 ttl=63 time=24.2 ms

--- 192.168.28.40 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 24.208/24.208/24.208/0.000 ms
root@sdvpn:~# 
```
and the traceroute
```
root@sdvpn:~# traceroute 192.168.40.40
traceroute to 192.168.40.40 (192.168.40.40), 30 hops max, 60 byte packets
 1  10.0.0.40 (10.0.0.40)  21.349 ms  22.337 ms  22.576 ms
 2  tcpc040.abc.com (192.168.40.40)  22.565 ms  22.551 ms  22.541 ms
root@sdvpn:~# traceroute 192.168.28.40
traceroute to 192.168.28.40 (192.168.28.40), 30 hops max, 60 byte packets
 1  10.0.0.28 (10.0.0.28)  25.481 ms  30.117 ms  32.086 ms
 2  dcpc040.abc.com (192.168.28.40)  33.811 ms  35.360 ms  36.769 ms
root@sdvpn:~#
```



#### additonal steps
##### enable firewall NAT in each nodes router
not necessary , but if the wireguard node is behind a NAT router , then must enable NAT for wireguard

1.1.1.1 is the WAN IP of the router , and 192.168.80.4 is the wireguard LAN ip, I map port 224 to ssh and 12000 for wireguard
```
iptables -t nat -A PREROUTING -i eth1 -d 1.1.1.1 -p tcp --dport 224 -j DNAT --to-destination 192.168.80.4:22
iptables -t nat -A PREROUTING -i eth1 -d 1.1.1.1 -p udp --dport 12000 -j DNAT --to-destination 192.168.80.4:12000
```

#### summary

if want to add more nodes into VPN , just follow the logic and steps.
```
create private/public key
create wg0.conf 
add new nodes in every other nodes wg0.conf as peer
```


1. for route , must add remote network in AllowedIPs 
2. check ip_forward is enable 
3. I think the postup haws no effect here , because the firewall service was disable by default , and if I use iptables -F to flush all firewall rules , the network still remain in connected.
4. need to create an ansible playbook for this


#### Update

##### strongswan IPSEC VS wireguard

**wireguard almost twice faster than strongswan**

iperf test with wireguard VPN 30 seconds benchmark
```
root@sdvpn:~# iperf -c 192.168.40.7 -t 30
------------------------------------------------------------
Client connecting to 192.168.40.7, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
[  3] local 10.0.0.80 port 48270 connected with 192.168.40.7 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-30.1 sec  65.1 MBytes  18.1 Mbits/sec
root@sdvpn:~# 
```


iperf test with strongswan VPN
```
root@sdvpn:~# iperf -c 192.168.40.7 -t 30
------------------------------------------------------------
Client connecting to 192.168.40.7, TCP port 5001
TCP window size: 85.0 KByte (default)
------------------------------------------------------------
[  3] local 192.168.80.4 port 57806 connected with 192.168.40.7 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-30.1 sec  35.6 MBytes  9.94 Mbits/sec
root@sdvpn:~# 
```

