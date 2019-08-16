---
title: "[筆記] 用pxe 安裝系統，完成後送出郵件通知 / send mail notification after pxe install"
date: 2019-07-31T11:06:33+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: ['筆記']
tags: ['PXE','ubuntu','linux']
author: "Eric Chang"
---

最近有個任務，需要大量安裝client

想用PXE來處理，只要user開機按F12(acer 桌機) 選擇PXE Boot

然後選擇OS版本，就可以自動進行安裝

安裝完成後，會自動重新開機，接著就用ansible來做user環境設定

PXE的部份本來是沒有什麼問題，自動安裝系統的部份都做好了

可是因為這次的量比較多，想說讓每一台在完成PXE安裝後的第一次重開機

就送出一封郵件來通知我，說已經完成安裝，可以執行ansible 了

看似很簡單的一件事情，卻搞了我兩天....

<!--more-->

本來在 preseed 檔案中，就有 preseed/late_command 可以用

但是測試了很多遍，才終於找到正確的語法

```
d-i preseed/late_command \
        in-target apt-file update; \
        in-target passwd --expire root  ;\
        in-target /bin/sh -c 'echo "hostname|mail -s pxe_install_complete admin@abc.com" > /etc/rc.local'                                             
```

這會把目標主機上的 /etc/rc.local 的內容改成只有一行
```
hostname|mail -s pxe_install_complete admin@abc.com
```

這樣就可以讓主機在完成系統安裝後，第一次重新開機時，送出郵件通知

可是呢，因為ubuntu 開機時，本來就會去執行 /etc/rc.local

所以「每次」開機後，都會送出郵件通知

但是我只想要接到一次通知就好了啊

有文章說可以用 s6-svc 來處理

不過我沒弄懂怎麼用

另一個是用ansible來處理

又或者是，讓這個指令在送出郵件後，「自我還原」或者「自我更新」

自我還原的部份可以這樣做

```
hostname|mail -s pxe_install_complete admin@abc.com 
echo "#!/bin/sh -e\nexit 0" > /etc/rc.local
```

所以preseed 那邊的語法就要改一下
```
  in-target /bin/sh -c 'echo "hostname|mail -s pxe_install_complete admin@abc.com;\"exit 0\" > /etc/rc.local' 
```

這樣一來，在送出郵件後，/etc/rc.local 的檔案內容會被恢復成只有底下這一行
```
exit 0
```
暫時先這樣子處理

### 更新 ###

因為直接把 /etc/rc.local 的內容改掉，實在讓我有點不放心

所以想到一個方式，先備份 /etc/rc.local 然後加入我要的功能

因為我只需要它跑一次就好，所以就可以在最後面加入還原剛剛複製的備份檔案

簡單說在preseed 檔案中 改成這樣
```
d-i preseed/late_command \
        in-target apt-file update; \
        in-target passwd --expire root  ;\
        in-target cp /etc/rc.local /etc/rc.local.bak ;\
        in-target /bin/sh -c 'echo "hostname|mail -s pxe_install_complete admin@abc.com" > /etc/rc.local' ;\
        in-target /bin/sh -c 'echo "cp /etc/rc.local.bak /etc/rc.local" >> /etc/rc.local'
```

在開機之後，會先送出郵件通知，然後會把剛剛複製的備份覆蓋回來，變成原本的 rc.local

這樣就不會有什麼問題了


