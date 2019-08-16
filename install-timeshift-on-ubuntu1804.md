---
title: "Install Timeshift on Ubuntu1804"
date: 2019-03-11T14:02:30+08:00

noSummary: false
featuredImage: "https://h.cowbay.org/images/post-default-11.jpg"
categories: [筆記]
tags: [ubuntu,backup]
author: "Eric Chang"
---

最近要開始測試client安裝 ubuntu 18.04 的 ansible playbook

因為要不斷的修正，所以想到一直有在自己電腦上執行的timeshift這個軟體

可以很簡單快速的備份、恢復系統狀態

可是不知道為什麼，在ubuntu 18.04 上安裝就是會發生錯誤....

<!--more-->

因為client 的環境都躲在proxy後面，一開始我想說直接下
```
export http_proxy=http://proxy_server:port
export https_proxy=http://proxy_server:port
```
然後再去依照官方的指令，新增repository就好
```
sudo add-apt-repository ppa:teejee2008/ppa
```

結果當然不是我想的那麼簡單...

直接就跳錯誤出來了

```
2019-03-11 13:57:28 [mini@pc074 ~]$ sudo add-apt-repository ppa:teejee2008/ppa
Cannot add PPA: 'ppa:~teejee2008/ubuntu/ppa'.
ERROR: '~teejee2008' user or team does not exist.
```

翻了一下google，有找到解法，但沒找到原因，就先這樣吧...(不求甚解)

```
export http_proxy=http://proxy_server:port
export https_proxy=http://proxy_server:port
sudo -E add-apt-repository -y ppa:teejee2008/ppa
sudo apt isntall timeshift
```

裝好之後，看一下怎麼執行，其實很簡單，也不用特別去指定什麼路徑
```
2019-03-11 14:00:59 [minion@hqpc074 ~]$ sudo timeshift --create --comments "after ansible"
First run mode (config file not found)
Selected default snapshot type: RSYNC
Selected default snapshot device: /dev/sda1
------------------------------------------------------------------------------
Estimating system size...
Creating new snapshot...(RSYNC)
Saving to device: /dev/sda1, mounted at path: /
Synching files with rsync...
Created control file: /timeshift/snapshots/2019-03-11_14-01-01/info.json        
RSYNC Snapshot saved successfully (79s)
Tagged snapshot '2019-03-11_14-01-01': ondemand
------------------------------------------------------------------------------
2019-03-11 14:02:20 [mini@pc074 ~]$ 
```

這樣就備份完了，來測試一下備份是否正常，先隨便裝個本來沒裝的軟體
```
2019-03-11 14:02:20 [minion@hqpc074 ~]$ sudo apt install joe
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  joe
0 upgraded, 1 newly installed, 0 to remove and 310 not upgraded.
Need to get 509 kB of archives.
After this operation, 2137 kB of additional disk space will be used.
Get:1 http://tw.archive.ubuntu.com/ubuntu bionic/universe amd64 joe amd64 4.6-1 [509 kB]
Fetched 509 kB in 2s (254 kB/s)
Selecting previously unselected package joe.
(Reading database ... 150020 files and directories currently installed.)
Preparing to unpack .../archives/joe_4.6-1_amd64.deb ...
Unpacking joe (4.6-1) ...
Processing triggers for mime-support (3.60ubuntu1) ...
Processing triggers for desktop-file-utils (0.23-1ubuntu3) ...
Setting up joe (4.6-1) ...
update-alternatives: using /usr/bin/joe to provide /usr/bin/editor (editor) in auto mode
Processing triggers for man-db (2.8.3-2) ...
Processing triggers for gnome-menus (3.13.3-11ubuntu1) ...
2019-03-11 14:09:39 [minion@hqpc074 ~]$
```
確定 joe 已經安裝
```
2019-03-11 14:09:51 [minion@hqpc074 ~]$ joe --help
Joe's Own Editor v4.6

Usage: joe [global-options] [ [local-options] filename ]...

Global options:
    -[-]restore             Restore cursor mode
    -[-]regex               Standard or JOE regular expression syntax
    -[-]square              Rectangular region mode
    -[-]icase               Case insensitive search mode 
    -[-]wrap                Search wraps mode
    -[-]menu_explorer       Menu explorer mode
    -[-]menu_above          Menu above/below mode
    -[-]notagsmenu          Tags menu mode
    -[-]search_prompting    Search prompting mode
    -[-]menu_jump           Jump into menu mode
    -[-]autoswap            Autoswap mode 
    -[-]mid                 Center on scroll mode
    -left nnn               Left scroll amount
    -right nnn              Right scroll amount
    -[-]guess_crlf          Auto detect CR-LF mode
    -[-]guess_indent        Guess indent mode
    -[-]guess_non_utf8      Guess non-UTF-8 mode
    -[-]guess_utf8          Guess UTF-8 mode
    -[-]guess_utf16         Guess UTF-16 mode
    -[-]transpose           Transpose menus mode
    -[-]marking             Region marking mode
    -[-]asis                Display meta chars as-is mode
    -[-]force               Force last NL mode
    -[-]joe_state           Joe_state file mode
    -[-]nobackups           Disable backups mode
    -[-]nodeadjoe           Disable DEADJOE mode
    -[-]nolocks             Disable locks mode
    -[-]nomodcheck          Disable mtime check mode
    -[-]nocurdir            Disable current dir 
    -[-]break_hardlinks     Break hard links 
    -[-]break_links         Break links 
    -[-]lightoff            Auto unmark 
    -[-]exask               Exit ask 
    -[-]beep                Beeps 
    -[-]nosta               Disable status line 
    -[-]keepup              Fast status line 
    -pg nnn                 No. PgUp/PgDn lines 
    -undo_keep nnn          No. undo records 
    -[-]csmode              Continued search 
    -backpath sss           Path to backup files 
    -[-]floatmouse          Click past end 
    -[-]rtbutton            Right button 
    -[-]nonotice            Suppress startup notice
    -[-]noexmsg             Suppress exit message
    -[-]help_is_utf8        Help is UTF-8
    -[-]noxon               Disable XON/XOFF
    -[-]orphan              Orphan extra files
    -[-]helpon              Start editor with help displayed
    -[-]dopadding           Emit padding NULs
    -lines nnn              No. screen lines (if no window size ioctl)
    -baud nnn               Baud rate
    -columns nnn            No. screen columns (if no window size ioctl)
    -skiptop nnn            No. screen lines to skip
    -[-]notite              Suppress tty init sequence
    -[-]brpaste             Bracketed paste mode
    -[-]pastehack           Paste quoting hack
    -[-]nolinefeeds         Suppress history preserving linefeeds
    -[-]mouse               Enable mouse
    -[-]usetabs             Screen update uses tabs
    -[-]assume_color        Assume terminal supports color
    -[-]assume_256color     Assume terminal supports 256 colors
    -[-]joexterm            Assume xterm patched for JOE
    -xmsg sss               Exit message
    -aborthint sss          Abort hint
    -helphint sss           Help hint
    -lmsg sss               Left side status line format
    -rmsg sss               Right side status line format
    -smsg sss               Status command format
    -zmsg sss               Status command format EOF
    -keymap sss             Keymap to use
    -mnew sss               Macro to execute for new files
    -mfirst sss             Macro to execute on first change
    -mold sss               Macro to execute on existing files
    -msnew sss              Macro to execute when new files are saved
    -msold sss              Macro to execute when existing files are saved
    -text_color sss         Text color
    -help_color sss         Help color
    -status_color sss       Status bar color
    -menu_color sss         Menu color
    -prompt_color sss       Prompt color
    -msg_color sss          Message color

Local options:
    +nnn                    Start cursor on specified line
    -[-]overwrite           Overtype mode
    -[-]hex                 Hex edit display mode
    -[-]ansi                Hide ANSI mode
    -[-]title               Status line context display mode
    -[-]autoindent          Autoindent mode
    -[-]wordwrap            Word wrap mode
    -tab nnn                Tab width
    -lmargin nnn            Left margin 
    -rmargin nnn            Right margin 
    -indentc nnn            Indent char 
    -istep nnn              Indent step 
    -[-]french              French spacing mode
    -[-]flowed              Flowed text mode
    -[-]highlight           Syntax highlighting mode
    -[-]spaces              No tabs mode
    -[-]crlf                CR-LF (MS-DOS) mode
    -[-]linums              Line numbers mode
    -[-]hiline              Highlight cursor line
    -[-]nobackup            No backup mode
    -[-]rdonly              Read only 
    -[-]smarthome           Smart home key 
    -[-]indentfirst         To indent first 
    -[-]smartbacks          Smart backspace 
    -[-]purify              Clean up indents 
    -[-]picture             Picture mode 
    -syntax sss             Syntax
    -colors sss             Scheme 
    -encoding sss           Encoding 
    -type sss               File type 
    -[-]highlighter_context ^G uses highlighter context 
    -[-]single_quoted       ^G ignores '... ' 
    -[-]no_double_quoted    ^G ignores "... " 
    -[-]c_comment           ^G ignores /*...*/ 
    -[-]cpp_comment         ^G ignores //... 
    -[-]pound_comment       ^G ignores #... 
    -[-]vhdl_comment        ^G ignores --... 
    -[-]semi_comment        ^G ignores ;... 
    -[-]tex_comment         ^G ignores %... 
    -text_delimiters sss    Text delimiters 
    -language sss           Language 
    -cpara sss              Paragraph indent chars 
    -cnotpara sss           Non-paragraph chars 
2019-03-11 14:09:54 [minion@hqpc074 ~]$ 
```

然後還原到剛剛做的備份，用 --list 看一下
```
2019-03-11 14:11:08 [minion@hqpc074 ~]$ sudo timeshift --list
Device : /dev/sda1
UUID   : d0efcb3d-e04a-41b8-a046-55557499f4d3
Path   : /
Mode   : RSYNC
Device is OK
1 snapshots, 108.3 GB free

Num     Name                 Tags  Description    
------------------------------------------------------------------------------
0    >  2019-03-11_14-01-01  O     after ansible  

2019-03-11 14:11:17 [minion@hqpc074 ~]$ 
```

然後還原，中間會問你要不要重新安裝 grub ，就看個人需求，我是都會讓它重新安裝一次啦

```
2019-03-11 14:11:53 [minion@hqpc074 ~]$ sudo timeshift --restore --snapshot '2019-03-11_14-01-01' --target /dev/sda1


******************************************************************************
To restore with default options, press the ENTER key for all prompts!
******************************************************************************

Press ENTER to continue...

Re-install GRUB2 bootloader? (recommended) (y/n): y

Select GRUB device:

Num     Device  Description              
------------------------------------------------------------------------------
0    >  sda     ATA TS128GSSD370S [MBR]  
1    >  sda1     ext4, 128.0 GB GB       

[ENTER = Default (/dev/sda), a = Abort]

Enter device name or number (a=Abort): 0

******************************************************************************
GRUB Device: /dev/sda
******************************************************************************

======================================================================
WARNING
======================================================================
Data will be modified on following devices:

Device     Mount
---------  -----
/dev/sda1  /    


Please save your work and close all applications.
System will reboot after files are restored.

======================================================================
DISCLAIMER
======================================================================
This software comes without absolutely NO warranty and the author takes no responsibility for any damage arising from the use of this program. If these terms are not acceptable to you, please do not proceed beyond this point!

Continue with restore? (y/n): y
Mounted '/dev/sda1' at '/mnt/timeshift/restore/'
******************************************************************************
Backup Device: /dev/sda1
******************************************************************************
******************************************************************************
Snapshot: 2019-03-11_14-01-01 ~ after ansible
******************************************************************************
Restoring snapshot...
Synching files with rsync...

Please do not interrupt the restore process!
System will reboot after files are restored

.d..t...... etc/
>f.st...... etc/mailcap
.d..t...... etc/alternatives/
cLc.t...... etc/alternatives/editor -> /usr/bin/vim.gtk3
cLc.t...... etc/alternatives/editor.1.gz -> /usr/share/man/man1/vim.1.gz
cL+++++++++ etc/alternatives/editor.fr.1.gz -> /usr/share/man/fr/man1/vim.1.gz
cL+++++++++ etc/alternatives/editor.it.1.gz -> /usr/share/man/it/man1/vim.1.gz
cL+++++++++ etc/alternatives/editor.ja.1.gz -> /usr/share/man/ja/man1/vim.1.gz
cL+++++++++ etc/alternatives/editor.pl.1.gz -> /usr/share/man/pl/man1/vim.1.gz
cL+++++++++ etc/alternatives/editor.ru.1.gz -> /usr/share/man/ru/man1/vim.1.gz
.d..t...... home/minion/
.d..t...... mnt/
.d..t...... timeshift/
.d..t...... tmp/
.d..t...... usr/bin/
.d..t...... usr/share/
.d..t...... usr/share/applications/
>f.st...... usr/share/applications/mimeinfo.cache
.d..t...... usr/share/doc/
.d..t...... usr/share/man/fr/man1/
cL+++++++++ usr/share/man/fr/man1/editor.1.gz -> /etc/alternatives/editor.fr.1.gz
.d..t...... usr/share/man/it/man1/
cL+++++++++ usr/share/man/it/man1/editor.1.gz -> /etc/alternatives/editor.it.1.gz
.d..t...... usr/share/man/ja/man1/
cL+++++++++ usr/share/man/ja/man1/editor.1.gz -> /etc/alternatives/editor.ja.1.gz
.d..t...... usr/share/man/man1/
.d..t...... usr/share/man/pl/man1/
cL+++++++++ usr/share/man/pl/man1/editor.1.gz -> /etc/alternatives/editor.pl.1.gz
.d..t...... usr/share/man/ru/man1/
cL+++++++++ usr/share/man/ru/man1/editor.1.gz -> /etc/alternatives/editor.ru.1.gz
.d..t...... usr/share/menu/
.d..t...... var/cache/apt/
>f..t...... var/cache/apt/pkgcache.bin
.d..t...... var/cache/apt/archives/
.d..t...... var/cache/apt/archives/partial/
.d..t...... var/cache/man/
>f..t...... var/cache/man/index.db
.d..t...... var/cache/man/cs/
.d..t...... var/cache/man/da/
.d..t...... var/cache/man/de/
.d..t...... var/cache/man/el/
.d..t...... var/cache/man/es/
.d..t...... var/cache/man/fi/
.d..t...... var/cache/man/fr.ISO8859-1/
.d..t...... var/cache/man/fr.UTF-8/
.d..t...... var/cache/man/fr/
>f..t...... var/cache/man/fr/index.db
.d..t...... var/cache/man/hr/
.d..t...... var/cache/man/hu/
.d..t...... var/cache/man/id/
.d..t...... var/cache/man/it/
>f..t...... var/cache/man/it/index.db
.d..t...... var/cache/man/ja/
>f..t...... var/cache/man/ja/index.db
.d..t...... var/cache/man/ko/
.d..t...... var/cache/man/nl/
.d..t...... var/cache/man/oldlocal/
.d..t...... var/cache/man/pl/
>f..t...... var/cache/man/pl/index.db
.d..t...... var/cache/man/pt/
.d..t...... var/cache/man/pt_BR/
.d..t...... var/cache/man/ro/
.d..t...... var/cache/man/ru/
>f..t...... var/cache/man/ru/index.db
.d..t...... var/cache/man/sk/
.d..t...... var/cache/man/sl/
.d..t...... var/cache/man/sr/
.d..t...... var/cache/man/sv/
.d..t...... var/cache/man/tr/
.d..t...... var/cache/man/zh/
.d..t...... var/cache/man/zh_CN/
.d..t...... var/cache/man/zh_TW/
.d..t...... var/lib/apt/
>f..t...... var/lib/apt/extended_states
.d..t...... var/lib/dpkg/
>f..t...... var/lib/dpkg/lock
>f.st...... var/lib/dpkg/status
>f.st...... var/lib/dpkg/status-old
.d..t...... var/lib/dpkg/alternatives/
>f.st...... var/lib/dpkg/alternatives/editor
.d..t...... var/lib/dpkg/info/
>f..t...... var/lib/dpkg/triggers/Lock
.d..t...... var/lib/dpkg/updates/
.d..t...... var/lib/misc/
>f..t...... var/lib/update-notifier/dpkg-run-stamp
>f.st...... var/log/alternatives.log
>f.st...... var/log/auth.log
>f.st...... var/log/dpkg.log
>f.st...... var/log/syslog
.d..t...... var/log/apt/
>f.st...... var/log/apt/eipp.log.xz
>f.st...... var/log/apt/history.log
>f.st...... var/log/apt/term.log
>f..t...... var/log/journal/8268f4544e414a37ab92123151d94126/system.journal
>f..t...... var/log/journal/8268f4544e414a37ab92123151d94126/user-1001.journal
.d..t...... var/log/timeshift/
*deleting   etc/joe/shell.sh
*deleting   etc/joe/shell.csh
*deleting   etc/joe/rjoerc
*deleting   etc/joe/jstarrc
*deleting   etc/joe/jpicorc
*deleting   etc/joe/joerc.zh_TW
*deleting   etc/joe/joerc
*deleting   etc/joe/jmacsrc
*deleting   etc/joe/jicerc.ru
*deleting   etc/joe/ftyperc
*deleting   etc/joe/editorrc
*deleting   etc/joe/
*deleting   etc/alternatives/editorrc
*deleting   usr/bin/rjoe
*deleting   usr/bin/jstar
*deleting   usr/bin/jpico
*deleting   usr/bin/joe
*deleting   usr/bin/jmacs
*deleting   usr/share/joe/syntax/yaml.jsf
*deleting   usr/share/joe/syntax/xml.jsf
*deleting   usr/share/joe/syntax/whitespace.jsf
*deleting   usr/share/joe/syntax/vhdl.jsf
*deleting   usr/share/joe/syntax/verilog.jsf
*deleting   usr/share/joe/syntax/typescript.jsf
*deleting   usr/share/joe/syntax/troff.jsf
*deleting   usr/share/joe/syntax/tex.jsf
*deleting   usr/share/joe/syntax/tcl.jsf
*deleting   usr/share/joe/syntax/swift.jsf
*deleting   usr/share/joe/syntax/sql.jsf
*deleting   usr/share/joe/syntax/spec.jsf
*deleting   usr/share/joe/syntax/sml.jsf
*deleting   usr/share/joe/syntax/skill.jsf
*deleting   usr/share/joe/syntax/sieve.jsf
*deleting   usr/share/joe/syntax/sh.jsf
*deleting   usr/share/joe/syntax/sed.jsf
*deleting   usr/share/joe/syntax/scala.jsf
*deleting   usr/share/joe/syntax/rust.jsf
*deleting   usr/share/joe/syntax/ruby.jsf
*deleting   usr/share/joe/syntax/rexx.jsf
*deleting   usr/share/joe/syntax/r.jsf
*deleting   usr/share/joe/syntax/python.jsf
*deleting   usr/share/joe/syntax/puppet.jsf
*deleting   usr/share/joe/syntax/ps.jsf
*deleting   usr/share/joe/syntax/properties.jsf
*deleting   usr/share/joe/syntax/prolog.jsf
*deleting   usr/share/joe/syntax/powershell.jsf
*deleting   usr/share/joe/syntax/php.jsf
*deleting   usr/share/joe/syntax/perl.jsf
*deleting   usr/share/joe/syntax/pascal.jsf
*deleting   usr/share/joe/syntax/ocaml.jsf
*deleting   usr/share/joe/syntax/md.jsf
*deleting   usr/share/joe/syntax/matlab.jsf
*deleting   usr/share/joe/syntax/mason.jsf
*deleting   usr/share/joe/syntax/mail.jsf
*deleting   usr/share/joe/syntax/m4.jsf
*deleting   usr/share/joe/syntax/lua.jsf
*deleting   usr/share/joe/syntax/lisp.jsf
*deleting   usr/share/joe/syntax/json.jsf
*deleting   usr/share/joe/syntax/jsf_check.jsf
*deleting   usr/share/joe/syntax/jsf.jsf
*deleting   usr/share/joe/syntax/js.jsf
*deleting   usr/share/joe/syntax/joerc.jsf
*deleting   usr/share/joe/syntax/jcf.jsf
*deleting   usr/share/joe/syntax/java.jsf
*deleting   usr/share/joe/syntax/iptables.jsf
*deleting   usr/share/joe/syntax/ini.jsf
*deleting   usr/share/joe/syntax/htmlerb.jsf
*deleting   usr/share/joe/syntax/html.jsf
*deleting   usr/share/joe/syntax/haskell.jsf
*deleting   usr/share/joe/syntax/haml.jsf
*deleting   usr/share/joe/syntax/groovy.jsf
*deleting   usr/share/joe/syntax/go.jsf
*deleting   usr/share/joe/syntax/git-commit.jsf
*deleting   usr/share/joe/syntax/fortran.jsf
*deleting   usr/share/joe/syntax/filename.jsf
*deleting   usr/share/joe/syntax/erlang.jsf
*deleting   usr/share/joe/syntax/erb.jsf
*deleting   usr/share/joe/syntax/elixir.jsf
*deleting   usr/share/joe/syntax/dockerfile.jsf
*deleting   usr/share/joe/syntax/diff.jsf
*deleting   usr/share/joe/syntax/debian.jsf
*deleting   usr/share/joe/syntax/debcontrol.jsf
*deleting   usr/share/joe/syntax/d.jsf
*deleting   usr/share/joe/syntax/css.jsf
*deleting   usr/share/joe/syntax/csharp.jsf
*deleting   usr/share/joe/syntax/csh.jsf
*deleting   usr/share/joe/syntax/context.jsf
*deleting   usr/share/joe/syntax/conf.jsf
*deleting   usr/share/joe/syntax/comment_todo.jsf
*deleting   usr/share/joe/syntax/coffee.jsf
*deleting   usr/share/joe/syntax/cobol.jsf
*deleting   usr/share/joe/syntax/clojure.jsf
*deleting   usr/share/joe/syntax/c.jsf
*deleting   usr/share/joe/syntax/batch.jsf
*deleting   usr/share/joe/syntax/awk.jsf
*deleting   usr/share/joe/syntax/avr.jsf
*deleting   usr/share/joe/syntax/asm.jsf
*deleting   usr/share/joe/syntax/ant.jsf
*deleting   usr/share/joe/syntax/ada.jsf
*deleting   usr/share/joe/syntax/4gl.jsf
*deleting   usr/share/joe/syntax/
*deleting   usr/share/joe/lang/zh_TW.po
*deleting   usr/share/joe/lang/uk.po
*deleting   usr/share/joe/lang/ru.po
*deleting   usr/share/joe/lang/fr.po
*deleting   usr/share/joe/lang/de.po
*deleting   usr/share/joe/lang/
*deleting   usr/share/joe/colors/zenburn.jcf
*deleting   usr/share/joe/colors/zenburn-hc.jcf
*deleting   usr/share/joe/colors/xoria.jcf
*deleting   usr/share/joe/colors/wombat.jcf
*deleting   usr/share/joe/colors/solarized.jcf
*deleting   usr/share/joe/colors/molokai.jcf
*deleting   usr/share/joe/colors/ir_black.jcf
*deleting   usr/share/joe/colors/gruvbox.jcf
*deleting   usr/share/joe/colors/default.jcf
*deleting   usr/share/joe/colors/
*deleting   usr/share/joe/charmaps/klingon
*deleting   usr/share/joe/charmaps/
*deleting   usr/share/joe/
*deleting   usr/share/applications/jstar.desktop
*deleting   usr/share/applications/jpico.desktop
*deleting   usr/share/applications/joe.desktop
*deleting   usr/share/applications/jmacs.desktop
*deleting   usr/share/doc/joe/man.md
*deleting   usr/share/doc/joe/help.pl.txt
*deleting   usr/share/doc/joe/hacking.md
*deleting   usr/share/doc/joe/copyright
*deleting   usr/share/doc/joe/changelog.Debian.gz
*deleting   usr/share/doc/joe/README.old
*deleting   usr/share/doc/joe/README.md
*deleting   usr/share/doc/joe/README.Debian
*deleting   usr/share/doc/joe/NEWS.md
*deleting   usr/share/doc/joe/
*deleting   usr/share/man/man1/rjoe.1.gz
*deleting   usr/share/man/man1/jstar.1.gz
*deleting   usr/share/man/man1/jpico.1.gz
*deleting   usr/share/man/man1/joe.1.gz
*deleting   usr/share/man/man1/jmacs.1.gz
*deleting   usr/share/man/ru/man1/joe.1.gz
*deleting   usr/share/menu/joe
*deleting   var/lib/dpkg/info/joe.prerm
*deleting   var/lib/dpkg/info/joe.preinst
*deleting   var/lib/dpkg/info/joe.postrm
*deleting   var/lib/dpkg/info/joe.postinst
*deleting   var/lib/dpkg/info/joe.md5sums
*deleting   var/lib/dpkg/info/joe.list
*deleting   var/lib/dpkg/info/joe.conffiles
*deleting   var/lib/misc/editorrc

sent 122,966,173 bytes  received 829 bytes  35,133,429.14 bytes/sec
total size is 7,201,040,584  speedup is 58.56

Re-installing GRUB2 bootloader...
Installing for i386-pc platform.
Installation finished. No error reported.

Updating GRUB menu...
Generating grub configuration file ...
Warning: Setting GRUB_TIMEOUT to a non-zero value when GRUB_HIDDEN_TIMEOUT is set is no longer supported.
Found linux image: /boot/vmlinuz-4.15.0-20-generic
Found initrd image: /boot/initrd.img-4.15.0-20-generic
Found memtest86+ image: /boot/memtest86+.elf
Found memtest86+ image: /boot/memtest86+.bin
done

Synching file systems...


Rebooting system...
Rebooting.

```

**要特別注意，restore完後，會自動reboot**

重新開機完成後，就入系統，執行看看 joe

```
2019-03-11 14:14:53 [minion@hqpc074 ~]$ joe

Command 'joe' not found, but can be installed with:

sudo apt install joe     
sudo apt install joe-jupp

2019-03-11 14:14:54 [minion@hqpc074 ~]$ 
```
可以看到，剛剛安裝的joe 又變成還沒安裝的狀態了，符合預期中的結果！

可以繼續測試playbook了！

