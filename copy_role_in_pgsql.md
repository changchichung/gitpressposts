---
title: "PostgreSQL 直接從已經存在的使用者複製權限到另一個使用者"
date: 2018-11-12T09:48:12+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-7.jpg"
categories: ['筆記']
tags: ['psql','筆記']
author: "Eric Chang"
---

因為工作上的需求，有個資料庫需要開放給不同team的人去存取

雖然都是在同一台機器上的同一個資料庫

但是希望能夠不同team的人用不同的資料庫使用者

這樣萬一出事，會比較好抓兇手？？

<!--more-->

總之呢，這個需求一直反覆出現

每次開發不同的AP，就要求一個不同的代號，實在有點煩

之前都是用pgadmin來處理，現在就改用psql來弄

其實也很簡單

```
CREATE ROLE b LOGIN;
GRANT a TO b;
```

可以帶入script ，用script 或者用ansible去跑就好了

省得每次都要在那邊手動改來改去..
