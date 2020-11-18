---
title: "ubuntu 20.04 install nvidia driver / CUDA / postgresql / pg_strom"
date: 2020-11-18T14:24:30+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-3.jpg
tags: [postgresql,nvidia,cuda,pg_strom,gpu]
author: "Eric Chang"
keywords:
  - postgresql
  - pg_strom
---

最近又開始在亂搞postgresql ，一直想要玩玩看GPU運算的威力，大概一年多前，有測試了 ubuntu 18.04 + postgresql + pg_strom ，可是當時因為pg_strom 不支援當時手邊的顯示卡，只好作罷。

Breaks here
<!--more-->
---
title: "ubuntu 20.04 install nvidia driver / CUDA / postgresql / pg_strom"
---

這次搞到一張GTX 1030 顯示卡，作業系統也升級到了 ubuntu 20.04 ，就再來弄一次看看

### 安裝 nvidia Driver

我還是選擇用 apt 新增ppa 的方式來安裝

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install ubuntu-drivers-common
sudo apt install nvidia-driver-450 
sudo reboot 
```

重開機後檢查一下是否有成功安裝

```
chchang@hqdc039:~/git/pg-strom$ lsmod|grep nvidia 
nvidia_uvm           1007616  2
nvidia_drm             49152  9
nvidia_modeset       1183744  11 nvidia_drm
nvidia              19722240  622 nvidia_uvm,nvidia_modeset
drm_kms_helper        184320  2 nvidia_drm,i915
drm                   491520  13 drm_kms_helper,nvidia_drm,i915
chchang@hqdc039:~/git/pg-strom$ 
```
OK ，看起來應該是沒有問題，接著來安裝 CUDA

### 安裝 CUDA

#### 下載 CUDA 安裝檔案
```
axel -n 10 http://developer.download.nvidia.com/compute/cuda/10.1/Prod/local_installers/cuda_10.1.243_418.87.00_linux.run
```

#### 執行安裝檔案進行安裝

注意在後面加上了 --override ，這是因為 ubuntu 20.04 預設的 gcc 是 9 ，但是 CUDA 目前還是只支援到 7 ，所以先用override 來解決這個問題，不然會出現 gcc version 的錯誤

```
sudo bash cuda_10.1.243_418.87.00_linux.run --override
```

安裝過程 nvidia 已經做成選單，就選擇要安裝的東西，記得<b>不要</b>選 Driver，因為剛剛已經安裝過了！

安裝完成後，需要修改一下 bashrc 
https://cyfeng.science/2020/05/02/ubuntu-install-nvidia-driver-cuda-cudnn-suits/

```
echo '# CUDA Soft Link' >> ~/.bashrc
echo 'export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}' >> ~/.bashrc
source ~/.bashrc
```
然後確認一下是不是正確安裝了
```
chchang@hqdc039:~/git/pg-strom$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243
chchang@hqdc039:~/git/pg-strom$ 
```

### 安裝 postgresql 

ubuntu 20.04 預設就是搭載 postgresql 12 ，所以安裝很方便
```
sudo apt install postgresql-12 postgresql-client-12 postgresql-client-common postgresql-client postgresql-common postgresql-contrib postgresql-server-dev-12
```

### 安裝 pg_strom

因為pg_strom 一樣也是不支援 gcc9 , g++9 ，所以先安裝會用到的套件
```
sudo apt install libicu-dev gcc-7 g++-7 libpmem-dev
```
然後改掉系統上的 gcc / g++

```
sudo unlink /usr/bin/gcc
sudo unlink /usr/bin/g++
sudo ln -s /usr/bin/gcc-7 /usr/bin/gcc
sudo ln -s /usr/bin/g++-7 /usr/bin/g++
```

然後clone pg_strom 回來做編譯， pg_config 的位置要看安裝的版本來決定
同時也要修改兩個檔案的link


```
sudo ln -snf /usr/lib/postgresql/12/lib/libpgcommon.a /usr/lib/x86_64-linux-gnu/libpgcommon.a
sudo ln -snf /usr/lib/postgresql/12/lib/libpgport.a /usr/lib/x86_64-linux-gnu/libpgport.a

git clone https://github.com/heterodb/pg-strom.git
cd pg-strom
make PG_CONFIG=/usr/lib/postgresql/12/bin/pg_config
sudo make install
```
這邊成功編譯之後，要來修改一下 postgresql，在 /etc/postgresql/12/main/postgresql.conf 中，加入底下這行

```
shared_preload_libraries = '$libdir/pg_strom'
```
然後重啟 postgresql service ，觀察一下syslog 有沒有錯誤
如果服務有起來，那基本上就安裝成功了

之後再來找看看有什麼測試pg_strom 的方式

