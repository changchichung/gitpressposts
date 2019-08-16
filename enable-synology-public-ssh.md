---
title: "筆記- 啟用群暉NAS (Synology NAS)的SSH Server  透過Publickey 認證免密碼登入"
date: 2018-11-05T14:16:54+08:00

featuredImage: "https://i.imgur.com/jcDQmI1.png"
noSummary: false
categories: ['筆記']
tags: ['筆記','synology','NAS','SSH']
author: "Eric Chang"
---

公司內有幾台NAS，其中有一台用來放開發人員的postgresql dump file
之前都是主要的開發人員上傳到google drive，分享出來 ，然後其他人去抓回來

這樣子有個問題是，當server要存取這些檔案時，就沒辦法了，除非透過一些 3rd party的軟體
像是這篇

https://www.omgubuntu.co.uk/2017/04/mount-google-drive-ocamlfuse-linux

或者是這篇

https://www.maketecheasier.com/mount-google-drive-ubuntu/

但是手邊的伺服器，原則上除非有必要，不然都沒有開放internet
所以導致明明檔案就在那邊，但是要取得就是很麻煩

<!--more-->

Dev_A upload to google drive ---> Dev_B Download from google drive ---> Dev_B scp download file to me ---> I upload to server.

有沒有？是不是很stupid (講話一定要烙英文)

既然有現成的NAS在那邊，幹嘛不用呢？(攤手)

聽說之前的人一直沒成功弄出來，讓Server可以直接去NAS存取檔案的方式，我記得這個不是很難啊
就順手整理一下

### 新增使用者帳號/ 確認家目錄存在

在NAS 的管理界面上新增一個帳號，假設叫 eric 好了

~~建立時，注意一下要指定家目錄路徑~~

更正： 群暉的界面好像不能指定家目錄

預設的路徑如下
```
eric:x:1071:100::/var/services/homes/eric:/sbin/nologin
```

不過我覺得怪怪的，因為在我手邊的幾台NAS底下 /var/services/homes 都切不過去
確認一下路徑，發現那個 ```@fake_home_link``` 根本就不存在啊！

```
admin@storage:/volume1$ ls -lart /var/services/homes 
lrwxrwxrwx 1 root root 24 May 23 14:14 /var/services/homes -> /volume1/@fake_home_link
admin@storage:/volume1$
```

我在想是不是之前的人有改過什麼..
anyway ，反正先不管這邊，直接修改 /etc/passwd檔案
```
sudo vim /etc/passwd
```
修正到正確的路徑，順便把shell 也改掉，不然不能登入

```
eric:x:1071:100::/volume1/homes/eric:/bin/sh
```

### 修改 /etc/ssh/sshd_config

再來修正預設沒有啟用 Publickey 驗證的 ssh

```
sudo vim /etc/ssh/sshd_config
```
確認底下三行存在

```
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys
```
### 將KEY傳到 NAS上

先建立相關目錄，順便修正一下目錄權限

```
chmod 755 /volume1/homes/eric
mkdir -p /volume1/homes/eric/.ssh
chmod 700 /volume1/homes/eric/.ssh
```

再來把Publickey 傳到NAS，複製貼上也好，ssh-copy-id也可以，同時修正權限
```
vim /volume1/homes/eric/.ssh/authorized_keys 
chmod 0600 /volume1/eric/.ssh/authorized_keys
```

### 重啟SSH

本來這個步驟應該可以用
```
synoservicectl --restart sshd
```
來解決
但是實際上這個指令只會把你踢出 SSH session ....( WTF!!! )

所以還是要去NAS的管理界面，去關閉再打開SSH (有點蠢..)
![Synology WEB UI](https://i.imgur.com/jcDQmI1.png)

然後就可以測試用Publickey 來登入NAS了

```
2018-11-05 14:47:12 [mini@s009 ansiblecontrol]$ ssh admin@storage 
admin@storage:~$ 
```
確認免密碼登入無誤了！




