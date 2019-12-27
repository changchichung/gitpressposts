---
title: "[筆記]在ansible中，取得loop register後的值/ Ansible Get Value From Loop Register"
date: 2019-12-27T09:09:05+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-7.jpg
tags: ['ansible']
author: "Eric Chang"
keywords:
  - ansible
---

今天在寫一支客製化 firefox 的playbook

因為firefox 會給每個user 建立一個由亂數字串組成的default profile

所以每個user的 default profile 都不同

也因此在用register處理的時候，碰到了一些問題

<!--more-->

### TASK 內容

其實task 內容沒有很複雜，簡單來說，就是先檢查user的 firefox profile 目錄

如果存在就pass，如果不存在就幫user建立

```
- name: check if user profile exists
  become_user: "{{ item.name }}"
  stat:
    path: "{{ firefox_home }}"
  register: profile_exists
  with_items: "{{ users }}"
```

但是在接下來的task在執行的時候，碰到了問題

ansible 提示說profile_exists 沒有 stat 屬性

用過 stat module 的，應該都知道結果會是 var_name.stat.xxxx 這樣的用法

所以可以肯定的是 stat 這個屬性一定在

可是也一定有什麼問題，造成取不到值

當然就是先debug 來看看(有些不重要的屬性，我就先拿掉了)

```
ok: [hqdc075] => {
    "profile_exists": {
        "changed": false,
        "msg": "All items completed",
        "results": [
            {
                "ansible_loop_var": "item",
                "changed": false,
                "failed": false,
                "invocation": {
                    "module_args": {
                        "checksum_algorithm": "sha1",
                        "follow": false,
                        "get_attributes": true,
                        "get_checksum": true,
                        "get_md5": false,
                        "get_mime": true,
                        "path": "/home/changch/.mozilla/firefox"
                    }
                },
                "item": {
                    "name": "changch"
                },
                "stat": {
                    "atime": 1577348614.335831,
                    "attr_flags": "e",
                    "attributes": [
                        "extents"
                    ],
                    "executable": true,
                    "exists": true,
                    "gid": 19000,
                    "inode": 925342,
                }
            },
            {
                "ansible_loop_var": "item",
                "changed": false,
                "failed": false,
                "invocation": {
                    "module_args": {
                        "checksum_algorithm": "sha1",
                        "path": "/home/administrator/.mozilla/firefox"
                    }
                },
                "item": {
                    "name": "administrator"
                },
                "stat": {
                    "atime": 1577349447.4605036,
                    "attr_flags": "e",
                    "attributes": [
                        "extents"
                    ],
                    "block_size": 4096,
                    "blocks": 8,
                    "charset": "binary",
                    "exists": false,
                }
            },
```

一開始是先注意到了為什麼裡面還有 item ?

再往上看，就發現多了一個 results 把原本的list 清單給包起來了

所以原本用 profile_exists.stat.exists 可以取得目錄是否存在(true/false)

但是現在因為多了一層 results ，所以變成取不到值

沒關係，既然知道了有多了一層 results ，那就知道要怎麼做了

### 修正後續task

後面的task 稍微改一下，在指定 with_items 的時候，直接指定 profile_exists.results

接下來的用法就比較簡單了，跟原本接stat 變數的語法差不多


```
- name: run firefox to create default profile
  become_user: "{{ item.item.name }}"
  shell: "firefox --headless &"
  ignore_errors: True
  when: item.stat.exists  == false
  with_items: "{{ profile_exists.results }}"
```

透過這次的playbook 又知道了怎麼承接 loop register 跑出來的值，又學了一課

本來一開始的想法是register dynamic variable name 

像是 a_exist,b_exist,c_exist 這樣

不過ansible 不這樣處理，會自動的把所有結果都包在一個 results list 裡面

也算是還滿好用的啦


