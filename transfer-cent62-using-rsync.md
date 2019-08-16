---
title: "[筆記] 用rsync 移轉 centos 6.2的老機器 Transfer Cent6.2 using rsync"
date: 2019-03-27T17:44:49+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-9.jpg"
categories: ['筆記']
tags: ['centos','linux']
author: "Eric Chang"
---

公司的一台老伺服器空間不足了，要執行指令都會中斷，所以想要擴充空間。

看起來不難搞，事實上.....
<!--more-->

因為前人的功績，所以這台當初的磁碟分割有點「特別」！

實體硬碟是兩顆500G SATA

然後各切三個 partition (sda1/sda2/sda3 and sdb1/sdb2/sdb3)

sda2/sdb2 用250M 來組 raid1 (/dev/md0) 給 /boot 用

sda1/sdb1 各用40G 組raid1 (/dev/md1) 做成LVM 的VG ，然後再切出四個大小不同的分割
```
* / (sysvg-root)
* /usr (sysvg-usr)
* /var (sysvg-var)
* /tmp (sysvg-tmp)

```

What the Fuck !?!? 然後最好笑的是，40G的空間，還留了2G，可以讓後面的倒楣鬼(對，就是我)

在碰到空間不足的問題時，還可以「擴充」


然後剩下的sda3/sdb3 大概有四百多G空間，一樣是組raid1 然後分割給這些目錄用

```
* /data (datavg-datatest) (對的，沒有錯 真的叫 datavg-datatest，媽的你test個鬼啊！還在test的東西直接上線使用？)
* /web (datavg-web)
* /www (datavg-www)
* /home (datavg-home)
```

幹！誰可以告訴我 www/web 的差異在哪裡？為什麼要特別切兩個分割給這兩個目錄用？

快下班了，明天再來寫...

繼續補坑..

先說結論，最後成功的作法如下

* 加裝一顆512G SSD 到機器上
* 安裝作業系統(centos 6.2) 到 512G SSD，裝完後用SSD開機，進入系統後，先補完一些套件(lvm2,rsync等等)
* 在系統內直接偵測原先的raid (mdadm --assemble --scan)
* 執行 vgchange -ay 啟動 lvm，原來的lv 會出現在 /dev/mapper/xxxxxx
* 先備份 /etc , /usr  (usr 應該可以不用備份)
* mount /dev/mapper/xxxxxx /mnt，然後把資料抄到SSD的相對路徑裡面 sysvg-usr-->usr sysvg-var-->var datavg-datatest-->data (幹，再罵一次，test個什麼鬼！)
* 都抄完之後，把剛剛備份的/etc_backup 裡面的fstab 複製回 /etc ，這樣才能正常開機
* 重新開機，祈禱可以開起來
* 進入系統後，測試相關程式

這次的過程，踩了很多坑...

導致中間有一段時間一直在鑽牛角尖，想要把問題解掉，加上沒有適合的硬體，就連新買的 1TB SSD 也是地雷

所以花了三天才算成功

簡單紀錄一下碰到了哪些問題，以及對應的解決方式(如果有得解的話)..

### 加裝兩顆硬碟後，原系統無法使用

這個問題非常奇怪，因為空間不足嘛，所以很直接就想說加硬碟進去擴充空間好了。

結果加了兩顆500G SATA硬碟之後，原有的系統雖然可以開機，但是 SSH 沒有啟動

想說就登入檢查一下看看是什麼問題導致SSH起不來，然後呢，這邊就碰到第一個地雷了

# 沒有人有root 密碼！！！！

雖然有兩個帳號可以登入，但是都是一般user ，不能做 sudo 

可以做sudo的帳號，沒有人知道密碼(因為都是用key登入)

那用root 登入吧，結果手邊紀錄的密碼都是錯的

好吧，那進rescue mode 改密碼總可以吧？事情如果那麼簡單就好了！

進rescue mode 之後，要執行 passwd ，就會出現

```
 /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory
```

很奇怪吧，為什麼加硬碟進去，就會碰到這個問題？

這兩顆新增的硬碟之前是用來測試 zfs 的，所以上面有切了 zfs 的分割

就因為這樣所以會碰到這種詭異的狀況嗎？ 不知道，不確定，不想面對...

好，針對 bad ELF interpreter 的問題，網路上的解法都是重新安裝套件

BUT ..yes , there is always a "BUT"

yum install glibc.i686 一樣出現上面的錯誤，所以無法安裝....

很好，卡關...換下一招！

P.S 其實在這邊犯了個錯誤，既然新增硬碟進去出了問題

那應該移除這兩顆新增的硬碟就可以先恢復原來的狀態(可開機運作，只是要執行程式會空間不足)

<hr>

### clonezilla 來源是raid時，無法調整目標磁區的大小

因為上面失敗了，所以現在每次調整時，我都會用 clonezilla 先做一份磁碟的備份

但是因為clonezilla 在處理來源是raid 磁區時，是用dd 慢慢寫入

備份時間很久，因為原本的raid 已經切了99%的空間出去給LVM

所以clonezilla 看到的使用量就是99% ，也因此每次備份幾乎就是要用dd 寫完500G的資料

有興趣可以玩看看，在傳統硬碟上這樣做要花多少時間...

然後我每次都要複製兩顆出來...

在使用clonezilla 的過程中，突然想到clonezilla 可以動態調整目標磁區大小

前提是目標磁碟空間比來源空間大

這個功能要進入 advanced mode 才會看到

興沖沖的改用這個選項複製.....結果還是一樣的大小...

其實這個結果不算意外啦，算是一個小嘗試，如果有支援就算賺到

沒有支援也就算了..


<hr>

### raid擴充「成功」，但不是預期的容量

磁碟對拷完，改用對拷的磁碟開機

然後再加入兩顆 500G 的SATA硬碟 (怪的是，這次加兩顆硬碟，就不會造成系統SSH無法啟動)

經過一番手腳 (簡單說就是 mdadm 搭配 ```--add``` , ```--remove``` , ```--grow``` , ```--raid-devices``` 這幾個參數下去跑)

終於成功把新的硬碟給加入原有的RAID了

但是一檢查容量， WTF !! 為什麼還是跟原本一樣？

這邊就要說到 raid 的原理了，直接跳過，說結論

如果是 raid1(mirror) ，至少兩顆硬碟這部份沒有問題

但是不管你再加多少顆硬碟進去這個raid，他的容量「不會改變」「不會改變」「不會改變」

依照某個外國人的解釋，就是下面這樣
```
RAID 1 is pure mirroring.  No matter how many drives you add to RAID 1, the size never increases.  What you increase is how many drives can fail and how good your read performance is.
```

所以，在原地擴充現有的mirror 空間不可行！

下一招，擴充不可行，那我改raid type 可以吧？？

<hr>

### convert raid1 to raid5

既然raid1 直接加硬碟不可行，那我能不能加入硬碟之後，把raid1 轉成 raid5 呢？

理論上是可以的

BUT ...(沒錯，還是這個BUT)

我參考底下這篇，過程其實很順利

https://stevesubuntutweaks.blogspot.com/2015/06/converting-raid1-array-to-raid5-using.html

但是只要我動到放boot 那組RAID，重開機之後，就完全沒有畫面.....


不會載入 grub 來開機，所以我完全不知道發生了什麼事，也沒辦法進grub 改 root 位置

用rescue mode 進去想修grub ，都出現 unknown file system 

那不要動不就好了嗎？那這樣就要掛著一顆原本的舊硬碟來開機啊！什麼蠢方法啊！

所以，失敗！....

不過我覺得這個方式應該是比較可行的才對，或許會再找虛擬機來測試看看

<hr>

### migrate system using rsync

正所謂事不過三，前面都失敗三次了(已經花了兩天多的時間)

不能再這樣搞下去了

這次改用之前就用過的「正統」解決方式

* 卸除原本主機上的所有硬碟，並新增一顆512G 的SSD 
* 在新增的512G SSD上，安裝一樣的作業系統(CentOS 6.2)

```
  這邊也有地雷...不知道為什麼，CentOS 6.2 的ISO，我用easy2boot 作到隨身碟上
  安裝過程出現CDROM not FOUND的錯誤。好，那我用光碟片開機總可以吧
  BUT (又是你啊，BUT兄 )
  在安裝時一樣出現CDROM not FOUND , WTF !!
  最後是透過用網路安裝的方式
  指定URL到 http://vault.centos.org/6.2/os/x86_64 來進行安裝
  而且一開始因為6.2裝不起來，所以我改用centos 6.10 的ISO作到隨身碟進行安裝
  這樣子安裝就可以，可是最後從原本硬碟把資料抄回來後，會不能開機
  會有 /bin/awk too many links 類似這樣的錯誤訊息
  總之，還是要想辦法裝回一樣的作業系統

```

* 系統安裝完成，關機，接上原本主機的一顆硬碟，準備把資料抄回來

```
  如果會怕，可以用ubuntu 18.04 live DVD 開機，然後先安裝 mdadm
  接著執行 mdadm --assemble --scan 偵測原本的RAID
  這時候雖然會抓到RAID，但是因為上面是LVM 所以沒辦法直接掛進來
  需要再執行一次 vgchange -ay
  原本的lvm 磁區，就會出現在 /dev/mapper 裡面
  再執行 mount /dev/mapper/sysvg-usr /mnt 把目錄掛進來
```
* 分次掛載原本的目錄，然後sync 回SSD

```
  我是直接在系統內操作，沒有透過liveDVD
  先備份 /etc , /usr 
  然後就看原本LVM切了幾個，每一個都mount 進來，然後把資料用rsync 抄回對應的路徑底下就好
  抄完之後，把備份出來的 /etc.bak/fstab 複製回 /etc
  這樣才能正常開機
  
```

* 重開機

```
  正常的話，這時候重開機應該是可以進入系統
  接著就是請同事確認環境、指令是否可以執行
  然後修改一些像是目錄權限, uid,gid 等等的問題
  基本上系統這樣就轉完了
```

<hr>

### 地雷產品

在這過程中，因為本來想直接做線上升級到1TB的空間，所以買了兩個 TCELL TT550 960G SSD 來用

![TCELL TT550 960G SSD](https://i.imgur.com/94hPMJG.png)

結果因為線上升級失敗，改用rsync來做，所以就把這兩顆SSD用 mdadm 組成raid1 來做系統

安裝過程沒什麼問題，但是在從舊硬碟抄資料回來的時候，發現速度很慢，而且會有卡住的情況

https://www.youtube.com/watch?v=NOIyRFit64k

從影片中可以看到，一開始複製檔案時，速度會往下掉，然後又慢慢爬上來，中間還會卡住

另一台用的是創見 S370 512G SSD 一顆 (MLC) ，這台就不會有這種問題

但是用Crystal Disk Mark 去測試這顆SSD ，速度又是正常 (4xx/3xx)

不知道是不是因為做RAID的關係，或者是linux 的影響？

總之最後這兩顆SSD 就被我退貨了...

話說我跟TCELL似乎犯衝啊...

上次買的4K EVO 64G隨身碟，也是速度很慢，跑了一次RMA換新的回來才正常

這次的960G SSD ($2999) 又有這種問題

看來下次要好好「評估」了..

<hr>

### 測試raid1轉raid5擴充空間


前面有提到，如果沒有LVM的話，我應該可以很輕易的加入兩顆硬碟到原有的raid1，然後就直接擴充容量

不過操作失敗了，但是我一直覺得這個應該很容易解決才對

所以我開了一台VM來做測試

OS: Ubuntu 18.04 Server x64
HDD: 16G vmdisk x4

一開始安裝時，就在安裝程式內設定一組raid1 (/dev/vda, /dev/vdb)，然後把系統灌在這個raid device上

安裝完成後，先進入系統確認一下現在的狀態

![](https://i.imgur.com/KDix5IJ.png)

可以看到有一個raid1 的磁區，由 /dev/vda1 , /dev/vdb1 組成，raid-devices:2 Array Size: 16765952 (15.99 GiB)

然後還有兩個磁碟沒有用到 (/dev/vdc , /dev/vdd)

接著為了進行模擬測試，先拍個快照，然後改用 ubuntu 18.04 Desktop 的ISO開機

進入系統後，要先開啟 terminal 來安裝 mdadm

```
apt install mdadm -y
```
![install mdadm in ubuntu 18.04 liveDVD](https://i.imgur.com/0tnEdW5.png)

然後把剛剛看到的兩顆還沒用到的磁碟切分割，加入RAID內

或者可以直接用sfdisk 從原有的磁碟抄partition table過來

我直接用 sfdisk 抄比較快，底下的指令是抄 vda 的partition table 到 vdc和 vdd

```
sfdisk -d /dev/vda | sfdisk /dev/vdc
sfdisk -d /dev/vda | sfdisk /dev/vdd
```

![sfdisk copt partition table](https://i.imgur.com/mlikld4.png)

然後檢查一下是不是正確
```
fdisk -l /dev/vdc
fdisk -l /dev/vdd
```

![check result after sfdisk](https://i.imgur.com/5qQNLbu.png)

再來就偵測原來的RAID，然後加入兩個剛剛做出來的分割到raid群組內

接著就直接把raid1轉成raid5，然後把raid-devices:2 改成 4，看一下狀態

可以看到，在把raid-devices 提升到4顆之後，原有的raid 就會開始進行reshape (不是rebuild唷)

```
mdadm --assemble --scan 
mdadm --add /dev/md0 /dev/vdc1 /dev/vdd1
mdadm --grow /dev/md0 --level=5
mdadm --grow /dev/md0 --raid-devices=4
cat /proc/mdstat
```

![convert raid1 to raid5 procedure](https://i.imgur.com/dxwQC0v.png)


耐心等待raid reshape 跑完，要一點時間，我只用16G的磁碟來測試也要跑個七八分鐘左右

不敢想像如果是幾TB的空間要跑多久...

等上面的程序跑完後，再看一下raid狀態，就會看到原有的RAID空間變大了，磁碟變多了，心情變好了，考試也都考100分了！

![check raid status after reshape](https://i.imgur.com/7E2qL8Q.png)

本來想說已經完成了，興沖沖的把raid 掛進來，看一下空間，卻還是16G ?????

```
mount /dev/md0p1 /mnt
df -h
```

![raid space still 16G](https://i.imgur.com/ILjDN1g.png)

其實這邊我一直沒搞懂，文章都說你就跑 e2fsck 檢查一次，然後跑 resize2fs 放大就結束了

問題是，我這樣做沒用...

![raid not expand after e2fsck and resize2fs](https://i.imgur.com/ij2AGw7.png)

最後我是直接請出 gparted 來做

叫出 gparted 之後，可以直接看到剛剛增加的空間沒有被放到原有的raid

那就直接 resize吧

![unallocated space in gparted](https://i.imgur.com/qLQMRAe.png)

gparted resize 太簡單了...直接用拉的就好了

![resize partition in gparted](https://i.imgur.com/KAs3gwu.png)

好，都做完了，重開機見真章！看看到底有沒有成功？

![check result after reboot](https://i.imgur.com/uGzxA9t.png)

空間順利放大了，打完收工！

我想以後如果又碰到一樣的狀況，大概就會用最後這個方式處理吧！


