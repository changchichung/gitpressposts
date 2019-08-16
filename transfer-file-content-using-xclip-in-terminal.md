---
title: "Transfer File Content Using Xclip in Terminal"
date: 2019-05-17T12:18:54+08:00
draft: false

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['linux']
tags: ['linux']
author: "Eric Chang"
---

工作上常會需要用ssh登入遠端主機檢查LOG，有必要的時候，還要把log複製回本機來處理。

以前都是傻傻的用 scp 傳檔案

之前就記得有這個xclip/xsel 可以用，但是一直沒有弄清楚怎麼執行

早上研究了一下，順便做個筆記。


<!--more-->
### 1. ssh 要加上 -X ###
不然會出現 

```
Error: Can't open display: (null)
```

這種錯誤訊息

```
-X      Enables X11 forwarding.  This can also be specified on a per-host basis in a configuration file.

             X11 forwarding should be enabled with caution.  Users with the ability to bypass file permissions on the remote host (for the user's X
             authorization database) can access the local X11 display through the forwarded connection.  An attacker may then be able to perform activi‐
             ties such as keystroke monitoring.

             For this reason, X11 forwarding is subjected to X11 SECURITY extension restrictions by default.  Please refer to the ssh -Y option and the
             ForwardX11Trusted directive in ssh_config(5) for more information.
```

### 2. remote 主機要安裝 xclip / xsel ###
```
2019-05-17 10:12:20 [minion@hqs019 ~]$ sudo apt install xsel
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  xsel
0 upgraded, 1 newly installed, 0 to remove and 73 not upgraded.
Need to get 19.0 kB of archives.
After this operation, 47.1 kB of additional disk space will be used.
Get:1 http://ftp.tw.debian.org/ubuntu bionic/universe amd64 xsel amd64 1.2.0-4 [19.0 kB]
Fetched 19.0 kB in 0s (80.3 kB/s)
Selecting previously unselected package xsel.
(Reading database ... 161032 files and directories currently installed.)
Preparing to unpack .../xsel_1.2.0-4_amd64.deb ...
Unpacking xsel (1.2.0-4) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Setting up xsel (1.2.0-4) ...

2019-05-17 10:13:32 [minion@hqs019 ~]$ sudo apt install xclip
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  xclip
0 upgraded, 1 newly installed, 0 to remove and 73 not upgraded.
Need to get 17.5 kB of archives.
After this operation, 52.2 kB of additional disk space will be used.
Get:1 http://ftp.tw.debian.org/ubuntu bionic/main amd64 xclip amd64 0.12+svn84-4build1 [17.5 kB]
Fetched 17.5 kB in 1s (16.2 kB/s)
Selecting previously unselected package xclip.
(Reading database ... 161038 files and directories currently installed.)
Preparing to unpack .../xclip_0.12+svn84-4build1_amd64.deb ...
Unpacking xclip (0.12+svn84-4build1) ...
Setting up xclip (0.12+svn84-4build1) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
```

### 3.執行方式 

執行以下指令，就可以把遠端的檔案內容傳送到「系統剪貼簿」，在本機就可以直接貼上了


```
cat copy_neonexus.csv |xclip -selection clipboard
```
