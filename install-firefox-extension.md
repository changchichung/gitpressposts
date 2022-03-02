---
title: "[筆記] 定製firefox extensions / customize Firefox Extensions in ubuntu"
date: 2022-03-02T09:11:01+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-18.jpg
tags: ['ubuntu','firefox']
author: "Eric Chang"
keywords:
  - ubuntu
  - firefox 
---
記得幾年前，為了要研究怎麼在公司所有電腦上安裝firefox extension

還特別去寫了一個 ansible role 來用

https://github.com/changchichung/ansible-firefox-install-extensions

<!--more-->
兩三年過去了，mozilla 官方似乎還是沒有提出一個官方的文件來說明

應該怎麼進行系統層級的extension 安裝

網路google 到的資訊又都很老舊

其中一個說法要找出 extension 安裝檔( xpi ) 裡面的一個檔案叫做 install.rdf

在這裡面找到一個欄位叫做 em:id 然後把下載回來的xpi 檔案改名成 em:id 這個值.xpi

再把這個 xpi 複製到 /usr/share/mozilla/extensions/{ec8030f7-c20a-464f-9b0e-13a3a9e97384}

結果這個是2013/14 年左右的文章

現在的extension 已經沒有那個 install.rdf 了

總之在翻了很多網頁(雖然講得大概都差不多，都是錯誤/過時的)

在官方文件又遍尋不著正式說明的情況下

只好自己想辦法了

找了一台桌機，安裝新的系統和firefox ，先只安裝一個 extension 然後觀察系統內的檔案變化

接著再移除、再安裝 這樣反覆驗證

終於找到了現在的路徑在

```
/usr/lib/firefox/distribution/extensions
```

底下是昨天測試時，複製進去的xpi


```
chchang@hqs185:/usr/lib/firefox/distribution/extensions$ ls
{0ac04bdb-d698-452f-8048-bcef1a3f4b0d}.xpi  {b9acf540-acba-11e1-8ccb-001fd0e08bd4}.xpi          jid1-FBaMKxTifTSahQ@jetpack.xpi
{0edc63bd-87d0-4fe5-afb7-468c20ba5514}.xpi  {b9db16a4-6edc-47ec-a1f4-b86292ed211d}.xpi          passwordmanager@avira.com.xpi
{3d1fbee0-687c-4032-9815-0f4519252d44}.xpi  chrome-gnome-shell@gnome.org.xpi                    private-relay@firefox.com.xpi
{46551EC9-40F0-4e47-8E18-8E5CF550CFB8}.xpi  {d10d0bf8-f5b5-c8b4-a8b2-2b9879e08c5d}.xpi          s3download@statusbar.xpi
{506e023c-7f2b-40a3-8066-bc5deb40aebe}.xpi  easyscreenshot@mozillaonline.com.xpi                savepage-we@DW-dev.xpi
{531906d3-e22f-4a6c-a102-8057b88a1a63}.xpi  enhancerforyoutube@maximerf.addons.mozilla.org.xpi  Shortkeys@Shortkeys.com.xpi
{5610edea-88c1-4370-b93d-86aa131971d1}.xpi  firefox@tampermonkey.net.xpi                        staged
{8419486a-54e9-11e8-9401-ac9e17909436}.xpi  foxyproxy@eric.h.jung.xpi                           support@nowpush.app.xpi
{943b8007-a895-44af-a672-4f4ea548c95f}.xpi  fvdmedia@gmail.com.xpi                              uBlock0@raymondhill.net.xpi
{9AA46F4F-4DC7-4c06-97AF-5035170634FE}.xpi  jid0-ODIKJS9b4IT3H1NYlPKr0NDtLuE@jetpack.xpi
admin@fastaddons.com_GroupSpeedDial.xpi     jid1-BYcQOfYfmBMd9A@jetpack.xpi
chchang@hqs185:/usr/lib/firefox/distribution/extensions$ 
```

在瀏覽器上面看起來會是這樣
![](https://i.imgur.com/s6lKVYI.png)

然後關閉firefox ，把這些xpi 通通砍掉，再次開啟firefox

卻發現這些extension 都還在？

所以再找了一下系統的 xpi 檔案

發現這些xpi 已經被複製到 $HOME 底下了

![](https://i.imgur.com/FQvUVN6.png)

所以看來在 /usr/lib/firefox/distribution/Extensions 目錄下，就是所謂的  system-wide extensions

當使用者第一次開啟firefox ，系統就會自動從這個目錄複製這些extension 到 user 的 firefox profile 裡面

那只要把要給user的extension 都先安裝在測試帳號中，然後從測試帳號的firefox profile 把這些extension 

複製回 /usr/lib/firefox/distribution/extensions

應該就可以達到為所有使用者安裝擴充套件的目標了

