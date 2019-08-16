---
title: "Nice Du Report Tool Durep"
date: 2018-11-06T15:24:29+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-9.jpg"
categories: ['筆記']
tags: ['linux','du']
author: "Eric Chang"
---
最近在重新規劃前人留下的backup爛攤子
各個伺服器統一備份到一台backup storage
想說如果每天能夠看到backup storage的磁碟用量的話
就可以抓出備份空間成長速度、推估需要多大的磁碟空間
找了一些工具，結果發現 durep 這個 ubuntu 內建的工具
基本上可以滿足我的需求

<!--more-->
我的需求其實很簡單

1. 可以指定目錄"深度"
2. 可以用圖表的方式顯示目前用量
3. 每天寄出報表

來看一下 durep 執行的狀況
**如果只指定一層，那就是顯示該目錄底下的使用狀況**

```
2018-10-29 15:50:21 [minion@tps006 ~]$ sudo durep -td 1 -sd /file
[ /file   259.0G (0 files, 3 dirs) ]
 259.0G [############################# ] 100.00% Oct 25  2017 team/
   1.7M [                              ]   0.00% Oct 23 14:04 html/
 741.1K [                              ]   0.00% Jul 11  2016 team_commons/
2018-10-29 15:50:26 [minion@tps006 ~]$ sudo durep -td 1 -sd /file
```
如果指定兩層
就顯示包含下一層目錄的磁碟使用量 *好像廢話*


```
2018-10-29 16:14:23 [mini@s006 ~]$ sudo durep -td 2 -sd /file
[ /file   259.0G (0 files, 3 dirs) ]
 259.0G [############################# ] 100.00% Oct 25  2017 team/
     259.0G [##############################] 100.00% Oct  3 15:08 tp/
         0b [                              ]   0.00% Jul 11  2016 temporary/
   1.7M [                              ]   0.00% Oct 23 14:04 html/
     748.5K [############                  ]  43.04% Jun 22  2016 font-awesome/
     282.2K [####                          ]  16.23% Jun 22  2016 css/
     241.0K [####                          ]  13.86% Jun 22  2016 js/
     222.9K [###                           ]  12.82% Jun 22  2016 img/
     210.7K [###                           ]  12.11% Jun 22  2016 fonts/
      18.6K [                              ]   1.07% Oct 23 14:04 index.html
       8.5K [                              ]   0.49% Jun 22  2016 less/
       2.2K [                              ]   0.12% Jun 22  2016 Gruntfile.js
       1.7K [                              ]   0.10% Jun 22  2016 README.md
       1.2K [                              ]   0.07% Jun 22  2016 mail/
       1.1K [                              ]   0.06% Jun 22  2016 LICENSE
       652b [                              ]   0.04% Jun 22  2016 package.json
        12b [                              ]   0.00% Jun 22  2016 .gitignore
 741.1K [                              ]   0.00% Jul 11  2016 team_commons/
     709.6K [############################  ]  95.75% Oct 23 14:05 tp/
      31.5K [#                             ]   4.25% Oct 23 14:05 temporary/
2018-10-29 16:14:36 [mini@s006 ~]$ 
```

搭配mail 使用，有點可惜的是在郵件內顯示，格式稍微有點跑掉
不過至少需要看到的目錄總使用量(左上角)，還有各個子目錄的用量都可以清楚的看到
如果能夠有縮排就更好了！

![https://i.imgur.com/ySNJJWx.png](https://i.imgur.com/ySNJJWx.png)



