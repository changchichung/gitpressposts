---
title: "[筆記] 在ubuntu 18.04 下安裝nvidia 顯示卡驅動程式以及 pgstrom / Install Nvidia Driver Cuda Pgstrom in Ubuntu 1804"
date: 2019-08-20T14:51:54+08:00
noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-7.jpg"
categories: ['筆記']
tags: ['nvidia']
author: "Eric Chang"
keywords:
  - nvidia 
  - cuda
  - pgstrom 
---

因為老闆說要試試看用GPU 來跑postgresql 威力

手邊剛好有一張 geforce gt 720

一開始沒想太多，看到有這張卡的驅動程式，然後CUDA也有支援

就直接從桌機拔下來，接去LAB Server ，然後就開始一連串的難關了...

<!--more-->

整個過程大致上分為四個步驟

### 安裝 nvidia driver
### 安裝 CUDA
### 安裝 postgresql
### 安裝 pgstrom


************
#### 安裝 nvidia driver

試過幾種方法，最後還是覺得用apt來安裝比較妥當
先新增repository、update、裝driver 
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```
然後用這個指令
```
ubuntu-drivers devices
```
看一下現在的驅動程式狀態
```
administrator@hqdc032:~/pg-strom$ ubuntu-drivers devices
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001288sv0000174Bsd0000326Bbc03sc00i00
vendor   : NVIDIA Corporation
model    : GK208B [GeForce GT 720]
driver   : nvidia-driver-410 - third-party free
driver   : nvidia-driver-418 - third-party free
driver   : nvidia-340 - distro non-free
driver   : nvidia-driver-430 - third-party free recommended
driver   : nvidia-driver-390 - third-party free
driver   : nvidia-driver-415 - third-party free
driver   : xserver-xorg-video-nouveau - distro free builtin
```
我這張卡，可以裝到 430 的版本，
接下來就繼續安裝驅動程式、裝完之後重開機
```
sudo apt install nvidia-driver-430 
sudo reboot 
```
這時候，應該可以看到一些基本資訊
```
lsmod|grep nvidia 
```

大概長這樣
```
nvidia_uvm            798720  0
nvidia_drm             45056  3
nvidia_modeset       1093632  7 nvidia_drm
nvidia              18194432  258 nvidia_uvm,nvidia_modeset
drm_kms_helper        172032  1 nvidia_drm
drm                   401408  6 drm_kms_helper,nvidia_drm
ipmi_msghandler        53248  2 ipmi_devintf,nvidia
```

到這邊 nvidia 驅動程式安裝完成，接下來安裝 cuda

#### 安裝 CUDA
同樣採用官網下載deb 回來安裝的方法

到這邊 https://developer.nvidia.com/cuda-downloads 

依照自己的系統選擇

我選擇 Linux -- x86_64 -- Ubuntu -- 18.04 -- deb(local)

畫面上就會有安裝步驟，照著做就沒問題了
```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-1-local-10.1.243-418.87.00_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-1-local-10.1.243-418.87.00/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
```

一樣，安裝完成後重新開機，然後來編譯一個 deviceQuery 的小程式來看看資訊

```
cd /usr/local/cuda-10.1/samples/1_Utilities/deviceQuery
sudo make
```

會產生一個叫 deviceQuery 的執行檔，執行後，會有相關資訊

```
administrator@hqdc032:/usr/local/cuda-10.1/samples/1_Utilities/deviceQuery$ ./deviceQuery 
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GT 720"
  CUDA Driver Version / Runtime Version          10.1 / 10.1
  CUDA Capability Major/Minor version number:    3.5
  Total amount of global memory:                 1996 MBytes (2093416448 bytes)
  ( 1) Multiprocessors, (192) CUDA Cores/MP:     192 CUDA Cores
  GPU Max Clock rate:                            797 MHz (0.80 GHz)
  Memory Clock rate:                             900 Mhz
  Memory Bus Width:                              64-bit
  L2 Cache Size:                                 524288 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(65536), 2D=(65536, 65536), 3D=(4096, 4096, 4096)
  Maximum Layered 1D Texture Size, (num) layers  1D=(16384), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(16384, 16384), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  2048
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 1 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            No
  Supports Cooperative Kernel Launch:            No
  Supports MultiDevice Co-op Kernel Launch:      No
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 10.1, CUDA Runtime Version = 10.1, NumDevs = 1
Result = PASS
```

到這邊， CUDA 也安裝完成，再來是簡單的 postgresql 11 

#### 安裝 postgresql 11

步驟差不多，就是新增repository，然後選擇要安裝的套件，不過套件的選擇和平常安裝postgresql 不太一樣

```
sudo apt install wget ca-certificates
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
sudo apt update
sudo apt install postgresql-client-11 postgresql-11 postgresql-server-dev-11 postgresql-common libicu-dev
```

跑完之後，就直接啟動 postgresql service 就可以了

再來是最麻煩的 pgstorm

#### pgstorm

話說，這軟體到底叫啥名字？ pgstrom , pg-strom ? 看起來就像是拼錯字啊！ =.=

https://github.com/heterodb/pg-strom

先 git clone 回來，然後make、make install

講是很簡單，但是一開始碰到很多問題，有去github 跟開發團隊回報，幸好有回覆我..

總之，目前在ubuntu 18.04 + postgresql-11 的環境下編譯是沒有問題了

```
git clone https://github.com/heterodb/pg-strom.git
cd pg-strom
make && sudo make install
```

再來設定一下 postgresql

#### postgresql 相關設定

需要修改一下postgresql.conf，來指定載入 pgstrom 的 library

官方是這麼說的

```
PG-Strom module must be loaded on startup of the postmaster process by the shared_preload_libraries. Unable to load it on demand. Therefore, you must add the configuration below.
```
修改 /etc/postgresql/11/main/postgresql.conf
加入一行
```
shared_preload_libraries = '$libdir/pg_strom'
```

然後還有其他三個要修改，不過這個不改不會影響pgstrom 的啟動

看狀況要不要修改吧

```
max_worker_processes = 100
shared_buffers = 10GB
work_mem = 1GB
```

修改完後，重新啟動 postgresql service 有沒有很感動！？我終於可以享受用GPU跑SQL Query 的快感啦！！！

咦？？等等，為什麼postgresql service 沒起來！？

看一下 /var/log/syslog

```
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: Error: /usr/lib/postgresql/11/bin/pg_ctl /usr/lib/postgresql/11/bin/pg_ctl start -D /var/lib/postgresql/11/main -l /var/log/postgresql/postgresql-11-main.log -s -o  -c config_file="/etc/postgresql/11/main/postgresql.conf"  exited with status 1:
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: 2019-08-20 14:23:43.538 CST [11806] LOG:  PG-Strom version 2.2 built for PostgreSQL 11
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: 2019-08-20 14:23:43.565 CST [11806] LOG:  PG-Strom: GPU0 GeForce GT 720 - CC 3.5 is not supported
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: 2019-08-20 14:23:43.565 CST [11806] FATAL:  PG-Strom: no supported GPU devices found
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: 2019-08-20 14:23:43.565 CST [11806] LOG:  database system is shut down
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: pg_ctl: could not start server
Aug 20 14:23:43 hqdc032 postgresql@11-main[11801]: Examine the log output.
Aug 20 14:23:43 hqdc032 systemd[1]: postgresql@11-main.service: Can't open PID file /run/postgresql/11-main.pid (yet?) after start: No such file or directory
Aug 20 14:23:43 hqdc032 systemd[1]: postgresql@11-main.service: Failed with result 'protocol'.
Aug 20 14:23:43 hqdc032 systemd[1]: Failed to start PostgreSQL Cluster 11-main.
```

啊幹！pg-strom 不支援這張GT 720啦！

https://github.com/heterodb/pg-strom/wiki/001:-GPU-Availability-Matrix

簡單說，就是至少從 GTX 1080 起跳，其他都不用想了

幹，花了兩天的時間在弄這東西，結果明明一開始就應該要先檢查的相容列表卻沒有檢查...

好了，現在就看准不准我買一張 GTX 1080 來測試了....

只是這價格嘛...嗯咳，不是我該煩惱的問題了..

