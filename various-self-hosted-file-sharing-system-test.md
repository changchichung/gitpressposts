---
title: "[筆記] 幾種可以自建服務的 File Sharing 系統比較"
date: 2021-06-25T15:49:54+08:00
noSummary: false
categories: ['筆記']
image: /images/post-default-18.jpg
tags: []
author: "Eric Chang"
keywords:
  - key1
  - key2
---

感覺最近應該會用到類似這樣的功能，趁著最近比較閒一點

就把系統弄起來玩玩看，順便建立ansible 的playbook

<!--more-->

### linx-server

https://github.com/andreimarcu/linx-server

目前已經停止開發的樣子

有Docker 版本，裝起來很容易，用起來也不難

可以自行架設伺服器

可以上傳任意類型的檔案

可以直接線上分享文字

可以自定分享密碼

上傳界面可以鎖密碼，但是鎖了密碼之後，就沒辦法用命令上傳檔案(不知道怎麼帶KEY進去)

#### 不支援整個目錄上傳

#### 關於API 如何使用沒有一個完整的說明

#### 始終找不到怎麼建立API KEY

![](https://i.imgur.com/83WOenv.png)

 

在console 下，可以直接上傳並取得超連結

```shell
chchang@hqdc039:~/docker/linx-server$ cat linx-server.conf
bind = 0.0.0.0:7779
sitename = myLinx
siteurl = https://share.com.tw
maxsize = 4294967296
maxexpiry = 43200
selifpath = s
allowhotlink = false
remoteuploads = false
nologs = true
force-random-filename = true
cleanup-every-minutes = 5
basicauth = false
authfile = /data/authfile

chchang@hqdc039:~/docker/linx-server$ docker-compose up -d
Creating linx-server ... done
chchang@hqdc039:~/docker/linx-server$ ./linx-client README.md 
Copied https://share.com.tw/fyd81h81.md into clipboard!              
chchang@hqdc039:~/docker/linx-server$ 

```

 



### Psitransfer

https://github.com/psi-4ward/psitransfer

不支援command 上傳

有docker版本，架設容易

適合給一般使用者用，可以自行設定密碼、保存期限

比較特別的是下載的連結可以產生QRCODE 

上傳檔案的頁面也可以鎖密碼

![](https://i.imgur.com/UMnPo0W.png)



### pictshare

https://github.com/HaschekSolutions/pictshare

有docker版本，但是需要自己手動調整

不然調整過的config 都會被蓋掉

調整過後的docker-compose.yml 我放了一份到github 上

https://github.com/changchichung/docker-compose-pictshare

需要拿掉pictshare.sh 中，每次自動更新config的部分

雖然web UI 有點醜，但是基本上想要的功能都有了

可以用WEB傳，也可以用terminal 傳

不限制上傳的檔案類型

可以限制可以上傳的subnet

回傳的URL 也可以有副檔名，所以可以直接連結當作圖床

算是很不錯用的了

![](https://i.imgur.com/4ujMfRA.png)



#### upload in terminal

```shell
chchang@hqdc039:~/docker/pictshare$ pict ~/Downloads/images/IMG_20190717_092723.jpg 
https://share.com.tw/1dpobr.jpg
chchang@hqdc039:~/docker/pictshare$ 

```

就先決定用這個 <strong>pictshare</strong> 吧


### 另外推薦的工具 anypaste

https://github.com/markasoftware/anypaste

這個雖然不能自己建立服務，需要依賴internet 上已經存在的多個網站服務

像是 file.io imgur hastebin 等等

不過呢，如果不是那麼計較安全性，要上傳的檔案不介意丟在internet上公開

那真的很推薦這個指令，不用安裝有的沒的一大堆，anypaste 本身就是一個script 整合了各家服務的上傳指令

所以「理論上」 要修改也不是太難..

跑起來大概像這樣

```shell
chchang@hqdc039:~/docker/pictshare$ anypaste ~/Downloads/images/IMG_20190717_092723.jpg 
Current file: /home/chchang/Downloads/images/IMG_20190717_092723.jpg
Attempting to upload with plugin 'imgur'
################################################################################################################# 100.0%

Link: https://imgur.com/y0Suzjf
Direct: https://i.imgur.com/y0Suzjf.jpg
Edit: https://imgur.com/edit?deletehashD
Delete: https://imgur.com/delete/fNJ

Upload complete.
Sucessfully uploaded: '/home/chchang/Downloads/images/IMG_20190717_092723.jpg'
All files processed. Have a nice day!
chchang@hqdc039:~/docker/pictshare$ 
```
也是非常方便的一個工具，值得推薦！



