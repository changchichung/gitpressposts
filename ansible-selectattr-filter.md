---
title: "[筆記] 還是 Ansible Selectattr "
date: 2018-11-29T11:22:28+08:00

featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['ansible']
author: "Eric Chang"
---

在上一篇 [Ansible how to use 'list' in yaml file ](https://h.cowbay.org/post/ansible-selectattr/)

有提到怎麼用 with_items / set_fact 來取得在yaml 檔案中的清單

不過就是有點醜

<!--more-->

這兩天又修改了一下，不需要用 when 來指定條件，改成用 filter 來篩選資料

將list整理成我們需要的「部份」資料就好，而不是所有資料都塞進來

```yaml
- name: set dc_users
  tags:
    - dcusers
    - depot_folder
    - env
  set_fact:
    dc_users: "{{ item.users }}"
  with_items: "{{ teams|selectattr('name','equalto','dc')|map(attribute='users')|list }}"
```

有沒有比較「優雅」(自己說...

先把 teams 這個 var 抓進來，然後用 selectattr 這個filter 選出 name == dc 的資料

再將篩選後的資料，用 map 去抓出 users 這個屬性，最後轉成 list

這樣子就可以直接得到 users 了
