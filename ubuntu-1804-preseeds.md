---
title: "[筆記] ubuntu 18.04 preseeds "
date: 2020-04-08T16:20:33+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-3.jpg
tags: ['pxe','preseeds']
author: "Eric Chang"
keywords:
  - pxe
  - preseeds
---
ubuntu 18.04 預設移掉了 /etc/rc.local 的功能

變成要用 systemd 的方式來運作，可是有點難用…

紀錄一下步驟，再來研究怎麼整合到 preseed 裡面

<!--more-->

### 1. 建立 rc-local.service

#### sudo vi /etc/systemd/system/rc-local.service
```
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

### 2. 建立 rc.local.bak

這個檔案的作用是，我只需要client在PXE 裝完系統後的第一次開機會發通知信件

所以如果一直保留著 /etc/rc.local 的變動，就變成每次開機都會送出信件

因此，需要先保留原本的 rc.local

在送出通知信件之後，就用原來的 rc.local 蓋掉修改過的 rc.local

#### sudo vi /etc/rc.local.bak

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.


exit 0
```

### 3. 建立 rc.local

#### sudo vi /etc/rc.local


```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

hostname|mail -s pxe_install_complete changch@abc.com
cp /etc/rc.local.bak /etc/rc.local

exit 0
```

### 4. 修改 rc.local permission

```
sudo chmod +x /etc/rc.local
```

### 5. 啟用 rc-local service

```
sudo systemctl enable rc-local
```

---

ubuntu 18.04 preseeds files

````
# Title: Ubuntu 18.04 preseed.cfg
#
# File: templates/os-ubuntu-1804-amd64-preseed.cfg
# modified by Eric , 2019/07
### Localization
 
# Keyboard selection.
# Disable automatic (interactive) keymap detection.
#d-i console-setup/ask_detect boolean false
d-i keyboard-configuration/xkb-keymap select us
 
d-i console-setup/ask_detect boolean false
d-i console-setup/layoutcode string us
d-i keyboard-configuration/ask_detect boolean false
d-i keyboard-configuration/layoutcode string us
 
# The values can also be preseeded individually for greater flexibility.
d-i localechooser/preferred-locale string en_US.UTF-8
d-i localechooser/supported-locales en_US.UTF-8
d-i debian-installer/language string en
d-i debian-installer/country string US
d-i debian-installer/locale string en_US.UTF-8                                                                                                                  
d-i localechooser/continentlist string Asia

### Network
d-i netcfg/choose_interface select auto

### Mirror settings
# If you select ftp, the mirror/country string does not need to be set.
d-i mirror/country string manual
d-i mirror/http/proxy string {{ proxy_env }}
d-i mirror/http/hostname string {{ pxe_preseed_mirror }}
d-i mirror/http/directory string /ubuntu
d-i mirror/http/mirror select {{ pxe_preseed_mirror }}


### Hostname
#d-i netcfg/get_hostname string unassigned-hostname
#d-i netcfg/get_domain string unassigned-domain
d-i netcfg/get_hostname string ubuntu
d-i netcfg/get_domain string abc.com

### account
d-i passwd/root-login boolean false
d-i passwd/user-fullname string Adminstrator
d-i passwd/username string administrator
d-i passwd/user-password-crypted   password $6$random_salt$VaSwPia0/6XHIicZLTaDceuRo/f9A6V4WazuZF/lhgQOhRJcKPO5yZ/ZxtBWrAhlDZOQ7GI3s4bPr9485Shry.
d-i user-setup/allow-password-weak boolean true
d-i user-setup/encrypt-home boolean false
d-i passwd/user-default-groups string audio cdrom video sudo adm



### Clock & Timezone
d-i clock-setup/utc boolean true
d-i time/zone string {{ pxe_preseed_timezone }}
d-i clock-setup/ntp boolean true

#d-i tzconfig/gmt            boolean true
d-i tzconfig/choose_country_zone/Asia select Taipei
d-i tzconfig/choose_country_zone_single boolean true

# Disable that annoying WEP key dialog.
d-i netcfg/wireless_wep string ubuntu

### Partitioning
d-i partman-auto/method string regular
d-i partman-lvm/device_remove_lvm boolean true
d-i partman-md/device_remove_md boolean true
d-i partman-lvm/confirm boolean true
d-i partman-lvm/confirm_nooverwrite boolean true
d-i partman-auto/choose_recipe select default-disk-layout
 
# Or provide a recipe of your own...
d-i partman-auto/expert_recipe string                         \
      default-disk-layout ::                                      \
              10240 20480 -1 ext4                           \
                      $primary{ } $bootable{ }                \
                      method{ format } format{ }              \
                      use_filesystem{ } filesystem{ ext4 }    \
                      label{ root }                           \
                      mountpoint{ / }                         \
              .                                               \
              1024 2048 200% linux-swap                       \
                      method{ swap } format{ }                \
              .                                               \

d-i partman/default_filesystem string ext4
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true
d-i partman/mount_style select uuid

### Base system installation
d-i base-installer/kernel/image string linux-generic

### Apt setup
d-i apt-setup/restricted boolean true
d-i apt-setup/universe boolean true
d-i apt-setup/backports boolean true
d-i apt-setup/services-select multiselect security
d-i apt-setup/security_host string security.ubuntu.com
d-i apt-setup/security_path string /ubuntu


# Package selection (tasksel --list-tasks)
#tasksel tasksel/first multiselect server, openssh-server
tasksel tasksel/first multiselect standard, openssh-server, ubuntu-desktop

# Individual packages (python-minimal for Ansible)
d-i pkgsel/include string ssh net-tools python2.7 axel curl vim unzip zip apt-file lynx elinks sysstat ntp htop screen apt-transport-https wget curl git rsync postfix mailutils

# Update policy 
d-i pkgsel/upgrade select safe-upgrade
d-i pkgsel/update-policy select unattended-upgrades
popularity-contest popularity-contest/participate boolean false
d-i pkgsel/updatedb boolean true

## postfix preseeding
postfix postfix/main_mailer_type select Internet Site
postfix postfix/mailname string pxe.abc.com
postfix postfix/protocols select  ipv4

# Reporting
popularity-contest popularity-contest/participate boolean false

# Bootloader
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean true
d-i finish-install/keep-consoles boolean true
d-i finish-install/reboot_in_progress note
d-i cdrom-detect/eject boolean true


#### Advanced options
d-i preseed/late_command string \
in-target wget --no-proxy http://192.168.1.7/rc-local.service -O /etc/systemd/system/rc-local.service ;\
in-target wget --no-proxy http://192.168.1.7/rc.local.pxe -O /etc/rc.local.pxe ;\
in-target wget --no-proxy http://192.168.1.7/rc.local.bak -O /etc/rc.local.bak ;\
in-target chmod +x /etc/rc.local.pxe ;\
in-target chmod +x /etc/rc.local.bak ;\
in-target cp /etc/rc.local.pxe /etc/rc.local ;\
in-target systemctl enable rc-local ;\
in-target ln -s /usr/bin/python3.6 /usr/bin/python

```


