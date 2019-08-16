---
title: "[筆記] 為了保存log 用script 指令執行ansible / Another Way to Keep Ansible Log using script command"
date: 2019-08-05T16:24:40+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-10.jpg"
categories: ['ansible']
tags: ['ansible']
author: "Eric Chang"
---

之前為了能夠在執行完 ansible playbook 後，能有個log 可以看

所以在每次執行的時候，都要加入 tee 的指令

像是
```
ANSIBLE_CONFIG=/home/D/ansiblecontrol/ansible.cfg /usr/local/bin/ansible-playbook  /home/D/ansiblecontrol/playbook.user_client.yml --vault-password-file=/home/D/ansiblecontrol/vault.passwd -i /home/D/ansiblecontrol/inventory/production -f1 --limit tyuserclients |tee /tmp/tyuserclients.log
```

一直都是放在crontab 裡面執行，也就沒有去管他

反正也沒有人關心結果怎樣 (攤手

<!--more-->

後來發現有個指令叫 script 可以完整紀錄指令執行期間的console 畫面變化(包含了ansi color!)

剛剛測試了一下，發現的確是可以用，不過也沒感覺有特別好用的地方，就只是做個紀錄吧，說不定以後其他地方可以用得到

```
sciprt -c 'make EXTRA_ARGS="-i inventory/production --limit hqpc074" user_client' -f hqpc074.out'
```
結果長這樣
![](https://i.imgur.com/F9KFAWV.png)

就真的跟console 的畫面一樣

