---
title: "[筆記] 在ubuntu 18.04 環境下 安裝 it 850UM 讀卡機 展延自然人憑證 / install it 850UM Card Reader in Ubuntu 18.04"
date: 2020-02-19T16:22:38+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-9.jpg
tags: []
author: "Eric Chang"
keywords:
  - 讀卡機
  - ubuntu
---

早上忘了要幹什麼，去看到手上的自然人憑證到期日是今年的 4/17

想說快到期了，看看能不能線上申請展延

結果辦公室沒有Linux 可以用的讀卡機 

OOXX 咧，我們可是號稱全Linux 環境捏！

結果居然沒有對應的硬體！？

於是馬上敗了一台據說有支援 Linux 的 IT 850UM 讀卡機！

<!--more-->

這是購買的網頁截圖，廠商號稱有支援Linux

![](https://i.imgur.com/ddAYSVD.png)

下午到手之後，直接接上去，發現ubuntu 18.04 還的確真的能抓到

但是，這型號為什麼不太一樣啊？？？怎麼是 IT 500U ??

![](https://i.imgur.com/eZidz0h.png)

先不管，直接開自然人憑證的網頁看能不能抓到..

https://moica.nat.gov.tw/renewcert.html

當然，事情絕對沒有那麼簡單！

開啟網頁之後，發現完全抓不到讀卡機！

啊不是號稱支援 Linux ??

於是開始翻google 看要怎麼處理

看到了這篇

http://gholk.github.io/linux-iccard-ccid-compile.html

不過這篇主要是在說其他的讀卡機，倒不是 it 850UM

但是一法通、萬法通！

就去文章裡面提到的連結看看

https://ccid.apdu.fr/

同樣的，要從 git 下載比較新版的code 回來自己編譯

```
git clone --recursive https://salsa.debian.org/rousseau/CCID.git
cd CCID
./bootstrap
./configure
make
```

然後，就報錯了！ 

```
/home/changch/git/CCID/missing: 列 81: flex：命令找不到
WARNING: 'flex' is missing on your system.
         You should only need it if you modified a '.l' file.
         You may want to install the Fast Lexical Analyzer package:
         <http://flex.sourceforge.net/>
Makefile:958: recipe for target 'tokenparser.c' failed
make[2]: *** [tokenparser.c] Error 127
make[2]: Leaving directory '/home/changch/git/CCID/src'
Makefile:437: recipe for target 'all-recursive' failed
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/changch/git/CCID'
Makefile:369: recipe for target 'all' failed
make: *** [all] Error 2
```

少了個套件叫 flex，補安裝上去
```
sudo apt install flex
```

然後再跑一次

```
./bootstrap
./configure
make
sudo make install
```

跑完之後，興沖沖的就去剛剛那個自然人憑證的網頁刷新！

然後還是沒抓到讀卡機 XDDDD

認份點，重新開機吧

重開機之後，再開啟網頁，就可以選擇讀卡機了！

接下來是關於自然人憑證展延的碎碎念

在選好讀卡機、輸入個人資料(怪了，不是應該從卡片裡面讀出來嗎？)和PIN碼之後

按下確認，然後網頁就卡住了...

發現是因為會有彈跳視窗，被firefox 給攔下來，放行之後，再按一次確認

就會看到跳出來的視窗、出現「寫入憑證中」，沒多久就關閉

又出現一個視窗，出現「讀取憑證中」，也是沒多久就關閉

然後咧？然後就沒有然後了！！

視窗還在原地不動，沒有成功、沒有失敗的訊息

就是個發呆的網頁！什麼提示都沒有！  WTF ！！！！

想說有出現寫入憑證，應該OK了吧，關掉視窗再來一次，才發現日期已經展延成功了

![](https://i.imgur.com/Qe0Uksh.png)

真的是很糟糕啊！加個訊息提示很困難嗎？？




