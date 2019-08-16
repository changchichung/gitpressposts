---
title: "[ansible] 引用事先定義好的yaml檔裡面的變數 - Ansible Selectattr From List in Dictionary file"
date: 2019-07-01T09:06:12+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-7.jpg"
categories: ['Ansible']
tags: ['Ansible']
author: "Eric Chang"
---

在ansible中，關於如何引用自定義的變數，一直讓我很頭疼

尤其是有牽涉到從外部導入yaml檔案時，更是常常讓我不知道到底該怎麼抓出想要的變數

這次還是用selectattr 來處理，希望下次能夠記得...

<!--more-->

首先是導入的yaml檔案，內容長這樣


```
client_hosts:
  abc.com:
  - host: hqdc021
    ipv4: 192.168.11.21
  - host: hqdc022
    ipv4: 192.168.11.22
  - host: hqdc023
    ipv4: 192.168.11.23
  - host: hqdc024
    ipv4: 192.168.11.24
  - host: hqdc025
    iuser: True
    ipv4: 192.168.11.25
    user: [yangj]
  - host: hqdc026
    ipv4: 192.168.11.26
    user: [changp, joy]
  - host: hqdc027
    ipv4: 192.168.11.27
  xyz.com:
  - host: dc021
    iuser: True
    ipv4: 192.168.1.21
	user: [tim]
  - host: dc022
    ipv4: 192.168.1.22
  - host: dc023
    ipv4: 192.168.1.23
  - host: dc024
    ipv4: 192.168.1.24
  - host: dc025
    ipv4: 192.168.1.25
    user: [eric]
  - host: dc026
    ipv4: 192.168.1.26
    user: [erica]
  - host: dc027
    ipv4: 192.168.1.27



```
在playbook中，首先先導入這個檔案
```
- name: load client_host
  include_vars:
    file: client_hosts.yml
    name: ch
```

然後用這個剛剛導入的檔案，去做出想要的清單

像底下這個，就是指定了client_hosts的 abc.com 這個域名底下，iuser有被定義的資料，再轉成list

```
- name: get internet user list
  set_fact:
    iuser_list: "{{ ch['client_host']['abc.com']|selectattr('iuser','defined')|list }}"
```

然後就可以用來做condition了

```
- name: copy environment block to /etc/environment
  copy:
   content: |
    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"
    all_proxy="{{ proxy_env }}"
    http_proxy="{{ proxy_env }}"
    https_proxy="{{ proxy_env }}"
    no_proxy="localhost,127.0.0.1,192.168.1.1/16,.xyz.com,.abc.com"
   dest: /etc/environment
  when: item.ipv4 == ansible_default_ipv4.address
  with_items: "{{ iuser_list }}"
```

