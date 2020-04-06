---
title: "[筆記] 在 Ubuntu 1404 Preseed 加入開機後自動發郵件通知安裝完成"
date: 2020-04-06T10:21:41+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-5.jpg
tags: ['pxe','ubuntu']
author: "Eric Chang"
keywords:
  - pxe
  - ubuntu
---

這是之前做過的task，client透過pxe開機後，會自動安裝ubuntu 14.04

在安裝完成後，會發出郵件通知管理者已經安裝完成

可是某次ansible 更新之後，反而沒辦法安裝完成

這次順手修改一下，同時更新了ansible 的template

<!--more-->

###os-ubuntu-1404-desktop-preseed.cfg

```
### Localization
 
# Keyboard selection.
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
d-i mirror/http/hostname string free.nchc.org.tw
d-i mirror/http/directory string /ubuntu
d-i mirror/http/mirror select free.nchc.org.tw
 
### Hostname
d-i netcfg/get_hostname string ubuntu
d-i netcfg/get_domain string abc.com
 
### account
d-i passwd/user-fullname string Adminstrator
d-i passwd/username string administrator
d-i passwd/user-password-crypted password $6$random_salt$12332112332123123123
d-i user-setup/allow-password-weak boolean true
d-i user-setup/encrypt-home boolean false
d-i passwd/user-default-groups string audio cdrom video sudo adm
 
### Clock & Timezone
d-i clock-setup/utc boolean true
d-i time/zone string Asia/Taipei
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

### 200% memory as swap , other space mount at /
# Or provide a recipe of your own...
# 2x physical RAM as swap , remain goes to root in ext4 , no LVM 
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
# ssh-server for 14.04/16.04 , openssh-server for 18.04
tasksel tasksel/first multiselect ubuntu-desktop,ssh-server
 
# Individual packages (python-minimal for Ansible)
#d-i pkgsel/include string python-minimal
d-i pkgsel/include string net-tools ssh python2.7 axel curl vim unzip zip apt-file lynx elinks sysstat ntp htop screen apt-tra
nsport-https wget curl git rsync postfix mailutils

# Update policy 
#d-i pkgsel/update-policy select none
d-i pkgsel/upgrade select safe-upgrade
d-i pkgsel/update-policy select unattended-upgrades
popularity-contest popularity-contest/participate boolean false
d-i pkgsel/updatedb boolean true
 
## postfix preseeding
# General type of configuration? Default:Internet Site
# Choices: No configuration, Internet Site, Internet with smarthost,
#   Satellite system, Local only
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
in-target apt-file update; \
#in-target DEBIAN_FRONTEND=noninteractive apt-get install mailutils -y;\
in-target rm -rf /etc/apt/sources.list.d/ubuntu-esm-infra-trusty.list ; \
in-target cp /etc/rc.local /etc/rc.local.bak ;\
in-target /bin/sh -c 'echo "hostname|mail -s pxe_install_complete changch@abc.com" > /etc/rc.local' ;\
in-target /bin/sh -c 'echo "cp /etc/rc.local.bak /etc/rc.local" >> /etc/rc.local' ;\
in-target passwd --expire root 

```
