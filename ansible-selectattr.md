---
title: "[筆記] Ansible how to use 'list' in yaml file "
date: 2018-11-27T16:50:53+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-1.jpg"
categories: ['筆記']
tags: ['ansible','linux']
author: "Eric Chang"
---

這幾天在玩ansible 時，碰到一個問題

假如我有個yaml檔作為資料來源，檔名是 abc.yml

大概長這樣

```
    "teams": [
        {
            "chinese_name": "TEAM1",
            "description": "TEAM1",
            "gid": 10125,
            "location": [
                "hq"
            ],
            "name": "aa",
            "users": [
                "chen",
                "chou",
                "huani",
                "yey",
                "wa"
            ]
        },
        {
            "chinese_name": "TEAM2",
            "description": "TEAM2",
            "gid": 10126,
            "location": [
                "hq"
            ],
            "name": "bb",
            "users": [
                "chhiao",
                "chgc",
                "chy",
                "hsi",
                "li",
                "li",
                "chgchi"
            ]
        }
		]
		
```

<!--more-->

稍微整理一下，比較容易看
```
    "teams": [
        {
            "chinese_name": "TEAM1",
            "description": "TEAM1",
            "gid": 10125,
            "location": ["hq"],
            "name": "aa",
            "users": ["chen","chou","huani","yey","wa"]
        },
        {
            "chinese_name": "TEAM2",
            "description": "TEAM2",
            "gid": 10126,
            "location": ["hq"],
            "name": "bb",
            "users": ["chhiao","chgc","chy","hsi","li","chgchi"]
        }
		    ]
		
```

在 ansible playbook 中，我用 include_vars 把這個檔案叫進來
```
- name: load teams.yml
  tags: env
  include_vars:
    file: files/konwen/teams.yml
```
這時候在這個執行階段，就會有一個變數叫 teams 裡面有 chinese_name/description/gid 等等這些屬性

其中的 location/users 又是另外一個 list

那如果我想要用users這個清單中的id作為建立帳號的來源

我就可以用底下這段，先把 users 裡面的內容，指給 dc_users 這個 localvar

然後加上 when 的條件，限定只有 name == aa 的 users 才會被指定給 dc_users


```
- name: set aa_users
  tags: env
  set_fact:
    aa_users: "{{ item.users }}"
  when: item.name == 'aa'
  with_items: "{{ teams }}"
```

這樣子執行下來是沒有問題的，不過就是醜了點 XD

之後要抓 user 帳號時，就可以直接用 aa_users 來跑迴圈

```
- name: create folder for aa_users 
  tags: env
  file:
    path: "/tmp/{{ item }}"
    owner: "{{ item }}"
    group: "{{ item }}"
    state: directory
  with_items: "{{ aa_users }}"
```

很簡單的概念，因為一開始在 teams 這個 var 裡面

users 這個屬性是一個 list 

p.s 講話一定要參雜用英文單字，這樣看起來比較屌...

所以沒辦法直接在底下的create folder task 直接叫出來

如果直接叫 teams.users ，那會是一個清單
```
 ["chen","chou","huani","yey","wa"]
```

然後 ansible 也很厲害，就這個樣子，他還是會去忠實的執行建立目錄

所以在 /tmp 底下就會多出一個

> ["chen","chou","huani","yey","wa"]

這個樣子的目錄 ....

所以我的處理方式就是先把這個清單丟給一個local變數 ( aa_users )

後面的task再用這個 {{ aa_users }} 來跑，這樣子就 OK 了

雖說很簡單，但是卡了我一整天吶...

