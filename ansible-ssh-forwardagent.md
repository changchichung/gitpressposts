---
title: "[筆記] ansible 設定 ssh_args 開啟 ForwardX11 / config ansible ssh_args to enable forwardagent"
date: 2019-12-24T14:41:37+08:00
draft: false
noSummary: false
categories: ['筆記']
image: https://h.cowbay.org/images/post-default-14.jpg
tags: ['ansible','ssh','forwardx11']
author: "Eric Chang"
keywords:
  - ansible
  - ssh
  - forwardx11
  - forwardagent
---

正確來說，我不曉得到底怎麼「稱呼」這個 forwardx11 / forwardagent

總之就是在寫一隻ansible playbook 

目的是用來安裝、設定 firefox 

包含安裝 firefox addon

但是一開始在執行的時候，碰到了一些錯誤

<!--more-->

### 錯誤訊息

playbook 在執行時的錯誤訊息如下
```
TASK [firefox : Create profiles] *************************************************************************************************
Tuesday 24 December 2019  14:28:58 +0800 (0:00:00.067)       0:00:00.946 ****** 
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: Exception: b'Error: no DISPLAY environment variable specified\n'
fatal: [hqdc075]: FAILED! => {
    "changed": false,
    "rc": 1
}

MSG:

MODULE FAILURE
See stdout/stderr for the exact error


MODULE_STDOUT:

Traceback (most recent call last):
  File "/home/minion/.ansible/tmp/ansible-tmp-1577168938.839576-98315583350576/AnsiballZ_firefox_profile.py", line 102, in <module>
    _ansiballz_main()
  File "/home/minion/.ansible/tmp/ansible-tmp-1577168938.839576-98315583350576/AnsiballZ_firefox_profile.py", line 94, in _ansiballz_main
    invoke_module(zipped_mod, temp_path, ANSIBALLZ_PARAMS)
  File "/home/minion/.ansible/tmp/ansible-tmp-1577168938.839576-98315583350576/AnsiballZ_firefox_profile.py", line 40, in invoke_module
    runpy.run_module(mod_name='ansible.modules.firefox_profile', init_globals=None, run_name='__main__', alter_sys=False)
  File "/usr/lib/python3.6/runpy.py", line 208, in run_module
    return _run_code(code, {}, init_globals, run_name, mod_spec)
  File "/usr/lib/python3.6/runpy.py", line 85, in _run_code
    exec(code, run_globals)
  File "/tmp/ansible_firefox_profile_payload_7amnitoq/ansible_firefox_profile_payload.zip/ansible/modules/firefox_profile.py", line 119, in <module>
  File "/tmp/ansible_firefox_profile_payload_7amnitoq/ansible_firefox_profile_payload.zip/ansible/modules/firefox_profile.py", line 109, in main
  File "/tmp/ansible_firefox_profile_payload_7amnitoq/ansible_firefox_profile_payload.zip/ansible/modules/firefox_profile.py", line 88, in create
Exception: b'Error: no DISPLAY environment variable specified\n'



MODULE_STDERR:

Shared connection to 192.168.11.75 closed.



PLAY RECAP ***********************************************************************************************************************
hqdc075                    : ok=3    changed=1    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   

```

稍微google 一下，都說是要設定 ssh forwardagent 

所以翻了一下 ansible 的設定文件，看要怎麼做

發現可以用 ssh_args 加入-o xxxxxx 

可是又找不到 ssh 怎麼用這個 -o 

只好又回去找辣個男人問看看ssh的參數

```
     -o option
             Can be used to give options in the format used in the configuration file.  This is useful for specifying options
             for which there is no separate command-line flag.  For full details of the options listed below, and their possi‐
             ble values, see ssh_config(5).

                   AddKeysToAgent
                   AddressFamily
                   BatchMode
                   BindAddress
                   CanonicalDomains
                   CanonicalizeFallbackLocal
                   CanonicalizeHostname
                   CanonicalizeMaxDots
                   CanonicalizePermittedCNAMEs
                   CertificateFile
                   ChallengeResponseAuthentication
                   CheckHostIP
                   Ciphers
                   ClearAllForwardings
                   Compression
                   ConnectionAttempts
                   ConnectTimeout
                   ControlMaster
                   ControlPath
                   ControlPersist
                   DynamicForward
                   EscapeChar
                   ExitOnForwardFailure
                   FingerprintHash
                   ForwardAgent
                   ForwardX11
                   ForwardX11Timeout
                   ForwardX11Trusted
                   GatewayPorts
                   GlobalKnownHostsFile
                   GSSAPIAuthentication
                   GSSAPIDelegateCredentials
                   HashKnownHosts
                   Host
                   HostbasedAuthentication
                   HostbasedKeyTypes
                   HostKeyAlgorithms
                   HostKeyAlias
                   HostName
                   IdentitiesOnly
                  .....
                  .....
```

很長，就不全部列出來了

看到重點是 ForwardAgent / ForwardX11 了

但是真的不曉得怎麼區分這兩種

反正只有兩個，就 try and error 吧

在ansible 中修改inventory file ，在想要修改的 host 後面加入 ssh_args="-o ForwardAgent=yes"
```
hqdc075 ansible_host=192.168.11.75 ssh_args="-o ForwardAgent=yes"

```
或者 ansible.cfg

在[ssh_connection]區段中

加入
```
ssh_args="-o ForwardAgent=yes"
```

再跑一次 就看到正常執行了
```
PLAY RECAP ************************************************************************************************
hqdc075                    : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Tuesday 24 December 2019  14:37:37 +0800 (0:00:01.027)       0:00:17.879 ****** 
=============================================================================== 
firefox : Install extensions --- 10.92s
install related pip packages ---- 4.71s
firefox : Install user prefs ---- 1.03s
firefox : Create profiles ------- 0.49s
firefox : export display -------- 0.44s
firefox : Configure profiles ---- 0.10s
firefox : debug ex ---- 0.07s

```

這次的過程，順便了解了ssh 加入 -X 可以在ssh session 中執行遠端主機上的圖形界面程式

例如 
```
ssh -X username@hostname firefox
```
就可以透過ssh 執行遠端的firefox

如下圖

![](https://i.imgur.com/5xoDFRe.png)

很酷！

