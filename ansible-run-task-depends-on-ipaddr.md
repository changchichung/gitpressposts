---
title: "[ansible] 用 ip 位置判斷是否要執行task /ansible run task depends on ipaddr"
date: 2019-07-23T15:06:37+08:00
draft: false 

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-7.jpg"
categories: ['ansible']
tags: ['ansible']
author: "Eric Chang"
---

因為工作上的需要，要修改client端的 /etc/environment 檔案

在有權限使用proxy 服務的user的環境中，加入proxy 的設定

原本的清單中，有host/user/ip 這幾個值可以拿來判斷

proxy server 那邊是採用ip 來控制，所以這邊也跟著用 ip 來判斷要不要修改 /etc/environment


<!--more-->
原本的想法是這樣

在playbook中，有兩個 task

當user ip (ansible_default_ipv4.address) 在清單內 ( {{ iuser_list }} )時

會去加入一些文字到 /etc/environment

反之，則取消這一段文字

```
- name: get internet user list
  set_fact:
    iuser_list: "{{ ch['client_hosts']['abc.com'] |selectattr('iuser', 'defined')| list }}"

- name: add proxy to /etc/environment
  blockinfile:
    path: /etc/environment
    marker: "#{mark} ANSIBLE MANAGED BLOCK#"
    block: |
      all_proxy="{{ proxy_env }}"
      http_proxy="{{ proxy_env }}"
      https_proxy="{{ proxy_env }}"
      no_proxy="localhost,127.0.0.1,192.168.1.1/16,.abc.com,.def.com"
 when: item.ipv4 == ansible_default_ipv4.address
 with_items: "{{ iuser_list }}"

# remove proxy when user not in iuser_list 
- name: removeproxy from /etc/environment
  blockinfile:
    path: /etc/environment
  marker: "#{mark} ANSIBLE MANAGED BLOCK#"
    block: ""
  when: ansible_default_ipv4.address not in "item.ipv4"
  with_items: "{{ iuser_list }}"
```

先做出一個可以上internet 的 user list

內容大概長這樣

```
hwaddress: f4:4d:30:45:ee:6f', host: pc114', ipv4: 192.168.1.114', user: [liwa'], iuser: True
hwaddress: f4:4d:30:45:ef:aa', host: pc120', ipv4: 192.168.1.120', user: [wany'], iuser: True
```

然後判斷當client ip 在這個清單中時，就去修改，反之就刪除修改的部份

有權限上internet的電腦在一開始跑就卡關了，這兩個task 都會被執行到

不應該是這樣才對呀，光看when 條件，會覺得這兩個條件應該是互斥的，怎麼會同時成立呢？

後來想想

在第一個task中，因為是用 item.ipv4 == ansible_default_ipv4.address 去做比對，所以很正常的一直比對到有符合的資料，然後開始進行task

但是在第二個task中，用的是ansible_default_ipv4.address not in item.ipv4 ，於是第一筆資料就符合條件，於是也開始執行task

在邏輯上，這樣的判斷沒有錯，錯的是我那打結的頭腦....

那怎麼解決呢？

把原本清單中的 ipv4 另外整理成一個list ，然後再去比對client ip  有沒有在這個list 中

就會變成這樣

```
- name: get internet user ip list
  set_fact:
    iuser_ip_list: "{{ ch['client_hosts']['konwen.com'] |selectattr('iuser', 'defined')| map(attribute='ipv4')|list }}"

- name: add proxy to /etc/environment
  blockinfile:
    path: /etc/environment
    marker: "#{mark} ANSIBLE MANAGED BLOCK#"
    block: |
      all_proxy="{{ proxy_env }}"
      http_proxy="{{ proxy_env }}"
      https_proxy="{{ proxy_env }}"
      no_proxy="localhost,127.0.0.1,192.168.1.1/16,.def.com.tw,.abc.com"
  when: ansible_default_ipv4.address in iuser_ip_list

# remove proxy when user not in iuser_list 
- name: remove proxy from /etc/environment
  blockinfile:
    path: /etc/environment
    marker: "#{mark} ANSIBLE MANAGED BLOCK#"
    block: ""
  when: ansible_default_ipv4.address not in iuser_ip_list
```

因為只比對 ip ，所以結果就是一翻兩瞪眼，有在裡面就跑第一個task ，沒有就跑第二個

------------------------------------------------------------------

不過呢， proxy server 那邊的playbook 也弄好了， client 這邊也知道怎麼跑了

但是，讓user可以透過proxy server 存取internet 的簽呈還是一直沒有下來 ....

都什麼年代了，還有半數以上的client 無法存取internet 

我實在是想不透啊..

