---
title: "[筆記] 建立一個帶著走的 VIM 環境 Creating portable Vim environment"
date: 2018-12-07T15:19:47+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-8.jpg"
categories: ['筆記']
tags: ['vim']
author: "Eric Chang"
---

因為工作的關係，現在很多時間都花在VIM的操作上

所以之前花了滿多時間，調整出一個適合自己的VIM環境

原本的作法是把這個設定好的環境，丟到自己建立的gitea 上面

然後每到一台新的機器，就要去clone 下來

<!--more-->

BUT (對，就是這個BUT)

手邊的機器有滿多不能直接連接internet的(酷吧，都什麼年代了，還鎖internet)

所以常會碰到要做git clone 不能動，還要額外去指定 proxy server 這些設定，挺麻煩的

今天剛好看到這個

https://junegunn.kr/2014/10/creating-portable-vim-environment

裡面介紹的小程式，可以把現在使用的vim 環境，打包成一個持行檔

那只要把這個執行檔丟到要控制的機器

那麼每一台機器都會有相同的vim 環境

比起之前要先git clone 然後 plugininstall 的作法要更方便許多，而且操作很簡單

只要先抓下來那個檔案並且執行

```shell
bash <(curl -L https://raw.githubusercontent.com/junegunn/myvim/master/myvim)
```
就會在執行的目錄底下產生一個 vim.$(whoami) 的執行檔

這個就是可以帶著走的 vim環境

然後就把這個檔案丟到看哪一台server上，然後在其他機器上面去下載回來

就可以有完全一樣的 vim 環境了，超級方便的！

