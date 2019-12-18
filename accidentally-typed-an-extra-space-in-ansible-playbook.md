---
title: "[筆記] 在ansible playbook中不小心多打了一個空格 / Accidentally Typed an Extra Space in Ansible Playbook"
date: 2019-12-18T14:44:27+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-10.jpg
tags: ['ansible']
author: "Eric Chang"
keywords:
  - ansible
---

剛剛在跑一個修改過的playbook，卻發現一個詭異的狀況

在用template產生檔案之前，爲了避免錯誤，所以我先用 file module 去建立目錄

怪就怪在，建立目錄的task沒錯，但是要產生檔案時，卻出現了目的目錄不存在的錯誤

<!--more-->

原本的 playbook 大概長這樣

```
- name: make dconf/profile folder
  file:
    path: "{{ item }} "
    state: directory
    owner: root
    group: root
    mode: a+rx
  with_items:
    - /etc/dconf/profile
    - /etc/dconf/db/msb.d
  
- name: generate dconf/profile/user template
  template:
    src: dconf/profile/user.j2
    dest: /etc/dconf/profile/user
    owner: root
    group: root
  
### the name must be 00_msb_settings ??
- name: generate 00_msb_settings 
  template:
    src: dconf/db/msb.d/00_msb_settings.j2
    dest: /etc/dconf/db/msb.d/00_msb_settings
    owner: root
    group: root
```
執行時的 LOG

```
TASK [userdesktop-1804 : generate 00_msb_settings] *******************************************************************************
Wednesday 18 December 2019  14:36:26 +0800 (0:00:00.328)       0:02:20.653 **** 
fatal: [hqpc108.abc.com.tw]: FAILED! => {
    "changed": false,
    "checksum": "06439b9bba9698ce7813525a4523afce72faefe8"
}

MSG:

Destination directory /etc/dconf/db/msb.d does not exist

```

怪了，登錄遠端電腦看一下
```
administrator@ubuntu:/etc/dconf/db$ ls -alrt
total 28
drwxr-xr-x 2 root root 4096 Dec 18 12:22  ibus.d
drwxr-xr-x 2 root root 4096 Dec 18 14:27  gdm.d
-rw-r--r-- 1 root root 2730 Dec 18 14:27  ibus
-rw-r--r-- 1 root root  240 Dec 18 14:27  gdm
drwxr-xr-x 2 root root 4096 Dec 18 14:36 'msb.d '
drwxr-xr-x 5 root root 4096 Dec 18 14:36  .
drwxr-xr-x 4 root root 4096 Dec 18 14:37  ..
```

發現了怪異的狀況，那個 msb.d 被單引號包起來了，代表包含了一些特殊字元

複製貼上後，才注意到，原來最後多了一個空白。

回去看 playbook 內容

```
- name: make dconf/profile folder
  file:
    path: "{{ item }} "
    state: directory
    owner: root
    group: root
    mode: a+rx
  with_items:
    - /etc/dconf/profile
    - /etc/dconf/db/msb.d
```

問題就出在 path 這邊，在 }} 和 " 之間，多了一個空格

然後ansible就很忠實的重現了這個語法

所以建立了 "/etc/dconf/profile " 以及" /etc/dconf/db/msb.d  "

於是就造成了後面的錯誤。

把 playbook 裡面的 語法修正就好了。

第一次碰到這狀況，也不是太難解決，不過還是簡單做個筆記好了

不然都沒啥文章了，哈哈！

