---
title: "[筆記] 為了metabase 修改 firefox 開啟網頁時使用的預設語言 change the preferred language in firefox for metabase"
date: 2018-11-15T11:06:28+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-9.jpg"
categories: ['筆記']
tags: ['筆記','firefox','metabase']
author: "Eric Chang"
---

最近在測試metabase，記得幾個月前就有測試過

但是當時的界面和現在的樣子差很多，看樣子改版還滿勤勞的

所以這次改用docker來建立，根本五分鐘不到就建好了(挖鼻孔)

不過呢，很討厭的是，一進去就發現語系採用的是簡體中文


<!--more-->

來看看這張圖
![metabase的簡體中文界面](https://i.imgur.com/oK6WmtJ.png)

WHAT THE FUCK !!!

這是哪一國的翻譯？？我相信對岸人才濟濟，絕對不至於翻譯出這種結果來..

想當然爾，我認為這個問題可以暫時不管，反正進入系統後，再去使用者界面設定就好

BUT .. (對，又是這個他X的BUT)

使用者設置裡面根本沒什麼可以改！

![metabase超陽春的使用者設置](https://i.imgur.com/DBRE5J6.png)

對，沒錯，就只有這樣！！ 請容許我再罵一次 WHAT THE FUCK !!!

* * *

好吧，罵完就算了，還是要想辦法解決，於是切到管理界面，的確是有語言設置

然後，我要再罵第三次 WHAT THE FUCK !!!

![metabase管理界面](https://i.imgur.com/HCSf8Ba.png)

```
The default language for this Metabase instance.This only applies to emails, Pulses, etc. Users' browsers will specify the language used in the user interface.
```

簡單說，這邊的語言設定「不影響」使用者界面，界面使用的語系，由各個瀏覽器決定！

好，那我去修改firefox 的語言看看

開啟 firefox ，點選右上角的三橫槓，找到語言選項

我一開始改成這樣

![firefox settings](https://i.imgur.com/630BBJF.png)

結果不管重開幾次，開啟metabase還是那個殘體中文字

後來想說，啊，會不會是「英語」「美語」的差異？

所以我把順序改成

![firefox language settings for metabase](https://i.imgur.com/pfdSWaC.png)

然後再開 metabase ，結果就如我所願的，改成英文界面了

![metabase in English](https://i.imgur.com/rc6SLSC.png)

真的是比殘體中文好太多了..

另外，關於繁體中文的部份，也到metabase的官方論壇去留言了

就看看官方要不要處理這個問題...








