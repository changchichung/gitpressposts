---
title: "[筆記] inxi 蒐集詳盡的硬體資訊 / inxi Collect Detail Hardware Info"
date: 2019-04-23T15:28:56+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-10.jpg"
categories: ['筆記']
tags: ['linux','bsd','inventory']
author: "Eric Chang"
---

最近因為一直碰到硬碟故障的問題，算起來那一批同時購買的5X顆 seagate 2T硬碟，已經有一半以上故障返修了....

然後又因為一直沒有添購新的硬碟，只能用這些快過保/已過保的撐著

所以最近不斷的在更換機器內的硬碟，而且還沒有熱插拔！

也導致原本負責處理盤點資產的同事困擾，因為跟手邊的紀錄已經對不起來了

然後就變成要對資產的時候，需要一台一台登入，然後去下不同的指令，取得想要的硬體資訊，超級麻煩的！

<!--more-->

幾次之後，終於決定透過ansible來做這件事

一開始的想法很簡單，就用 lshw/dmidecode這些指令去做

可是因為手邊的機器有ubuntu 18.04/16.04/14.04 , Debian 9 , Proxmox (based on debian ) , CentOS , FreeNAS

而有些系統預設沒有 lshw / dmidecode (對，FreeNAS 就是說你)

所以變成要依照系統不同，去下不同的指令，雖然都是ansible在跑，但是看到playbook的內容就很煩啊！

然後就不小心讓我翻到了 inxi 這個指令，根本就是救星啊！

直接來看輸出的範例

![sample of inxi output](http://i.imgur.com/OSx9cnz.png)

有沒有，是不是很優！

而且簡單易懂，還能抓到同事想看的資料，像是廠牌、型號、序號、記憶體類型(DDR2/3/4)

所以馬上捨棄 lshw/dmidecode ，改用 inxi 來跑

ansible role 的內容也很簡單

就偵測完之後，把結果送出給設定好的收件人

只是因為系統不同，大致上要分成 ubuntu/debian/centos 以及 freebsd 兩種

所以同樣的task 要跑兩次，一個要帶sudo 一個不用帶

然後BSD系列的機器，在inventory 裡面要帶入 ansible_ssh_user 

就這樣，沒有什麼太困難的

```YAML
######### use inxi instead ##################
- name: copy inxi binary to remote Ubnutu/Debian
  become: yes
  become_method: sudo
  copy:
   src: inxi
   dest: /usr/local/bin/inxi
   mode: a+rx,u+rwx
  when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian" or ansible_distribution == "CentOS"
 
- name: copy inxi binary to remote FreeBSD
  copy:
   src: inxi
   dest: /usr/local/bin/inxi
   mode: a+rx,u+rwx
  when: ansible_distribution == "FreeBSD"
 
- name: run inxi to collect Ubuntu/Debian hardware info
  become: yes
  become_method: sudo
  shell: "/usr/local/bin/inxi -c -Dxx -C -m -Z"
  register: du_hw_info
  when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian" or ansible_distribution == "CentOS"
 
- name: run inxi to collect FreeBSD hardware info
  shell: "/usr/local/bin/inxi -c -Dxx -C -m -Z"
  register: bsd_hw_info
  when: ansible_distribution == "FreeBSD"
 
- name: set Ubuntu/Debian inventory file
  template:
   src: etc/inventory.txt.j2
   dest: "/tmp/{{ ansible_hostname }}_inventory.txt"
   mode: a+r,u+rw
  when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian" or ansible_distribution == "CentOS"
 
- name: set FreeBSD inventory file
  template:
   src: etc/freenas_inventory.txt.j2
   dest: "/tmp/{{ ansible_hostname }}_inventory.txt"
   mode: a+r,u+rw
  when: ansible_distribution == "FreeBSD"

- name: send inventory file via mail
  tags: mail
  mail:
    host: 192.168.11.173
    port: 25
    secure: starttls
    subject: "{{ ansible_hostname }} inventory file"
    from: ansible
    to: "{{ recipient }}"
    #body: "{{ mail_body.stdout_lines }}"
    attach: "/tmp/{{ ansible_hostname }}_inventory.txt"

```

inventory 內容
```
hqs01.abc.com ansible_ssh_host=192.168.11.1                                                                                                         
hqs210.abc.com 
hqs230.abc.com 
hqs231.abc.com 
hqs234.abc.com 
hqs03.abc.com
hqs020.abc.com
hqs019.abc.com
hqs010.abc.com
hqs05.abc.com
hqs173.abc.com
###BSD Hosts ###
hqs099.abc.com ansible_ssh_host=192.168.11.99 ansible_ssh_port=22 ansible_ssh_user=root
hqs202.abc.com ansible_ssh_host=192.168.11.202 ansible_ssh_port=22 ansible_ssh_user=root
bbs089.abc.com ansible_ssh_host=192.168.0.89 ansible_ssh_user=root
```

ansible 又發揮了一次，另外，感覺這個指令可以用來寫資產管理系統耶...威力強大

而且又不用管作業系統是什麼，反正有執行檔，直接派過去 remote 端就好了！

真是讓我相見恨晚啊！



