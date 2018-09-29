---
layout: post
title: An incomplete Linux handbook
subtitle: 一份不完整的Linux生存手册
image: /ico/icon-linux.png
#bigimg: /img/path.jpg
tags: [linux]
---

[TODO]: <> (改善文字质量)
[TODO]: <> (实现中英双语)
[TODO]: <> (增加引用)

这份Linux生存手册记录了一些Linux系统的桌面用户在安装使用过程中可能遇到的问题及其解决方法。该手册将始终是“incomplete”的状态，因为它将会随着时间推移不断扩充和完善。由于内容繁多，推荐使用目录中的链接直接查看需要的内容。

[1. Linux installation & settings 系统安装与设置](#1)

+ [1.1 制作系统启动盘](#dd)
+ [1.2 GRUB2引导区设置](#grub2)
+ [1.3 系统分区自动挂载（fstab）](#fstab)
+ [1.4 挂载加密分区](#luks)
+ [1.5 减少swap写入](#reduce-swap)
+ [1.6 打开和关闭交换分区](#swap-on-off)
+ [1.7 开机后以系统为父进程自动执行命令](#startup)
+ [1.8 修改CentOS软件源](#cent-source)

[2. Linux customisation & fixing 系统定制与修复](#2)

+ [2.1 显示所有开机自动启动项目](#show-startup)
+ [2.2 锁定笔记本触控板](#lock-touchpad)
+ [2.3 ubuntu中合上屏幕后无操作](#lid)
+ [2.4 自定义命令](#command)
+ [2.5 禁用CentOS7的IPv6和防火墙](#cent-ipv6)
+ [2.6 向sudo group中添加用户](#sudoers)
+ [2.7 设置文件默认打开方式](#apps)
+ [2.8 启用nautilus上一级目录的BackSpace快捷键](#backspace)
+ [2.9删除ubuntu unity桌面lenses](#lenses)
+ [2.10 更改Gnome桌面默认文件夹地址](#folders)
+ [2.11 gnome ibus输入法设置](#ibus)
+ [2.12 手动创建Gnome桌面快捷方式](#desktop)
+ [2.13 修复gvfs-metadata大量消耗内存](#gvfs)

[3. Linux commands & tools 命令与工具](#3)

+ [3.1 查看系统信息](#catsystem)
+ [3.2 crontab计划任务](#crontab)
+ [3.3 rsync文件同步](#rsync)
+ [3.4 通过代理服务器使用yum和dnf](#proxy)
+ [3.5 maven local repository](#maven)
+ [3.6 向Gnome桌面发送通知](#gnome-notify)
+ [3.7 ssh免密码登录](#ssh-pwd)
+ [3.8 ssh反向隧道](#ssh-tunnel)
+ [3.9 samba共享文件夹设置](#samba)
+ [3.10 apache2和php5网站](#php5)
+ [3.11 将视频转换为mp4格式](#ffmpeg)

<h2 id='1'> 1. Linux installation & settings 系统安装与设置 </h2>

<h3 id='dd'> 1.1 制作系统启动盘 </h3>

使用dd将已下载好的linux系统镜像[os-image].iso写入到优盘上：

```
sudo dd if=[os-image].iso of=/dev/sdx bs=1M
```

优盘的设备文件为/dev/sdx，其中编号x可使用fdisk查询：

```
sudo fdisk -l
```

----------------------------------------------------------------

<h3 id='grub2'> 1.2 GRUB2引导区设置 </h3>

GRUB2包含3部分文件：
- /etc/default/grub    grub的自定义文件
- /etc/grub.d/         各操作系统的启动脚本
- /boot/grub2/grub.cfg  最终生成的主配置文件(或/boot/grub/grub.cfg)

/etc/grub.d/目录中默认包含以下文件：
- 00_header
- 10_linux
- 20_memtest86+
- 30_os-prober
- 40_custom

GRUB2 的/usr/sbin/grub2/grub2-mkconfig会根据/etc/default/grub文件和/etc/grub.d/中的脚本文件生成/boot/grub2/grub.cfg(或/boot/grub/grub.cfg)

#### 为linux添加grub启动项

在/etc/grub.d/中添加启动脚本11_linux：
```
#!/bin/sh -e
echo "Adding my custom Linux to GRUB 2"
cat << EOF
menuentry "My custom Linux" {
set root=(hd0,5)
linux /boot/vmlinuz
initrd /boot/initrd.img
}
EOF
```
menuentry一行为显示在GRUB2启动界面上的名称，set一行用来标示启动的硬盘和分区，特别要注意与GRUB不同的是，在GRUB2中硬盘的分区是从1开始计数的，而非GRUB中的0。但硬盘编号依然是从0开始计数的。例如(hd0,5)标示第1块硬盘上的第5个分区。

#### 为windows添加grub启动项

在/etc/grub.d/中添加启动脚本12_windows：
```
#!/bin/sh -e
echo "Adding Windows 7 to GRUB 2 menu"
cat << EOF
menuentry "Windows 7" {
set root=(hd0,1)
chainloader (hd0,1)+1
}
EOF
```

#### 生成/boot/grub2/grub.cfg

为添加的启动脚本添加执行权限：
```
sudo chmod a+x /etc/grub.d/11_linux
sudo chmod a+x /etc/grub.d/12_windows
```

生成/boot/grub2/grub.cfg：
```
sudo /usr/sbin/grub2/grub2-mkconfig --output=/boot/grub2/grub.cfg
```
注意此处若不添加output参数则会将生成的grub.cfg输出在默认输出设备（即屏幕）上。

将grub设置写入硬盘分区：
```
sudo grub-install /dev/sda
```

完成。重新启动：
```
sudo reboot
```

----------------------------------------------------------------

<h3 id='grub2'> 1.3 系统分区自动挂载（fstab） </h3>
以管理员账户修改/etc/fstab文件，自动挂载分区到/media/uraplutonium/Workstation，分区可以是ext4格式（line 13）、ntfs格式（line 15）或加密分区（line 17）。另外，若内存空间足够大，可以将临时文件夹/tmp挂载到内存中用以加速系统速度（line 19）。

“uid=1000”和“gid=1000”指定用户和用户组id，可用“id username”确认用户id。

{% highlight shell linenos %}
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sdb5 during installation
UUID=e4211512-aa99-466e-8184-83e856977c35 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sdb6 during installation
UUID=8e6b0a37-3367-4158-bcd8-a7480ca8f115 none            swap    sw              0       0
# mount the ext4 partition to Workstation
UUID=d48a5b68-881b-4f9e-8101-81499ea3e0c5 /media/uraplutonium/Workstation ext4 defaults,async,nosuid,nodev,nofail 0 2
# mount the NTFS partition to Workstation
UUID=64B08D642EA2D142	/media/uraplutonium/Workstation	ntfs-3g	defaults,locale=en_GB.UTF-8.zh_CN,uid=1000,gid=1000,dmask=022,fmask=022	0
# mount the encrypted partition to Workstation
/dev/disk/by-uuid/15dc4506-239e-416b-9969-b3e30e3835a9 /media/uraplutonium/Workstation auto nosuid,nodev,nofail,x-gvfs-show 0 0
# mount /tmp to RAM
tmpfs /tmp tmpfs defaults,exec,nosuid 0 0
{% endhighlight %}

<h3 id='luks'> 1.4 挂载加密分区 </h3>
要挂载加密的“crypt-luks”格式的加密ext4分区/dev/sdc1，执行以下指令以挂载至/mnt路径下：
```
sudo cryptsetup -v luksOpen /dev/sdc1 crypt
sudo mount /dev/mapper/crypt /mnt
```

----------------------------------------------------------------

<h3 id='reduce-swap'> 1.5 减少swap写入 </h3>

在/etc/sysctl.conf末尾加入下列内容，可减少swap的使用，以加快系统运行：

```
vm.swappiness=10
```

----------------------------------------------------------------

<h3 id='swap-on-off'> 1.6 打开和关闭交换分区 </h3>
打开和关闭所有交换分区的命令分别为swapon和swapoff：

```
sudo swapon -a
sudo swapoff -a
```

----------------------------------------------------------------

<h3 id='startup'> 1.7 开机后以系统为父进程自动执行命令 </h3>

有些命令或工具需要以系统为父进程启动，以[星际文件系统IPFS](https://ipfs.io/)和[去中心化云存储平台Sia](https://sia.tech/)为例，使用setsid命令启动：
```
setsid ipfs daemon
setsid /home/uraplutonium/Sia-v0.6.0-beta-linux-amd64/siad -d /home/uraplutonium/Sia
```

若需要开机后自动以系统为父进程自动启动后台进程，需要在/etc/rc.local中的exit 0之前插入以下内容：
```
su - uraplutonium -c "export IPFS_PATH=/media/uraplutonium/Extension/IPFS; setsid ipfs daemon"
su - uraplutonium -c "setsid /home/Sia-v0.6.0-beta-linux-amd64/siad -d /home/uraplutonium/Sia"
```
其中uraplutonium为管理员用户名。

----------------------------------------------------------------

<h3 id='cent-source'> 1.8 修改CentOS软件源 </h3>
先重命名原CentOS-Base软件源：
```
sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

然后，将网易163软件源CentOS6-Base-163.repo或sohu软件源CentOS-Base-sohu.repo文件放入/etc/yum.repos.d/，并执行：
```
sudo yum clean all
sudo yum makecache
```

----------------------------------------------------------------

<h2 id='2'> 2. Linux customisation & fixing 系统定制与修复 </h2>

<h3 id='show-startup'> 2.1 显示所有开机自动启动项目 </h3>
执行下列命令以在“Startup Applications Preferences”中显示隐藏的自动启动项目：
```
sudo sed -i 's/NoDisplay=true/NoDisplay=false/g' /etc/xdg/autostart/*.desktop
```

可能的默认启动项目包括：

+ AT SPI D-Bus Bus

AT SPI stands for Assistive Technology Service Provider Interface, a framework to integrate accessibility functions in applications. This command will create a new DBus for AT SPI specific functions.
Command: /usr/lib/at-spi2-core/at-spi-bus-launcher --launch-immediately

Status: unwanted until you need the accessibility features.

Reference: https://www.linuxfoundation.org/

+ AT SPI Registry

The AT SPI Registry is used by applications to interact with assistive technologies and peripherals.

Command: /usr/lib/at-spi/at-spi-registryd

Status: unwanted until you need the accessibility features.

Reference: http://www.linuxfromscratch.org/blfs/view/svn/gnome/at-spi.html

+ Bluetooth Manager

A GNOME classic applet for the GNOME panel to provide access to bluetooth devices configuration.

Command: bluetooth-applet

Status: unwanted until you use GNOME fallback session and you make use of the Bluetooth technology. Unneeded for GNOME Shell users.

+ Certificate and Key Storage

A GNOME Keyring agent which will provide access to your encryption certificates for applications.

Command: /usr/bin/gnome-keyring-daemon --start --components=pkcs11

Status: unwanted if you don’t use encryption certificates.

+ Disk Notifications

The Disk Utility notification is used to report disk failures using the SMART predictive technology.

Command: /usr/lib/gnome-disk-utility/gdu-notification-daemon

Status: recommended if your disks support the SMART, to discover incoming damages.

+ Evolution Alarm Notify

Alarm notifier for Evolution incoming events and appointments.

Command: /usr/lib/evolution/3.0/evolution-alarm-notify

Status: unwanted if you don’t use the Evolution calendar alarms.

+ Fcitx

Start Input Method

Command: fcitx-autostart

+ Files

The nautilus file manager for desktop icons.

Command: nautilus -n

Status: unwanted until you choose to enable the desktop with its icons.

+ GNOME Login Sound

Play a sound from the sound theme after the login to welcome the user. This is broken since years, I’ve checked in Fedora, Ubuntu, Debian and Arch Linux, don’t know why but every default sound theme seems to miss the required file.

Command: /usr/bin/canberra-gtk-play --id=”desktop-login”

Status: unwanted until you fix the default theme and wish to hear a welcome sound.

+ GNOME Settings Daemon

A daemon which grants access to all the GNOME system preferences.

Command: /usr/bin/gnome-settings-daemon

Status: recommended for everyone.

+ GPG Password Agent

A GNOME Keyring agent which will loads your GPG keys and allow you to insert the passphrases in a graphical window when a GPG key is required during signing.

Command: gnome-keyring-daemon --start --components=gpg

Status: unwanted if you don’t use GPG keys to sign or encrypt data.

+ GSettings Data Conversion

A conversion tool from GConf to GSettings, used to convert legacy settings into the new settings format used by GNOME 3.

Command: gsettings-data-convert

Status: wanted to migrate old schema configuration, its execution is really fast and light so that there’s no reason to disable it.

+ Mount Helper

Automount and autorun plugged devices

Command: /usr/lib/gnome-settings-daemon/gnome-fallback-mount-helper

+ Orca Screen Reader

The Screen reader for people with reading and sight difficulties.

Command: orca --no-setup --disable main-window --disable splash-window --disable magnifier --enable speech --enable braille

Status: unwanted until you need accessibility features for speech or braille.

+ PolicyKit Authentication Agent

An authentication agent which will require you user or administration password when applications need to check the user privileges. This doesn’t apply to sudo/su/gksu requests.

Command: /usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1

Status: recommended for everyone.

+ Power Manager

A GNOME daemon that communicates with the hardware for proper power management, stand by, suspend and reduce power consumption by underclocking the CPU.

Command: gnome-power-manager

Status: recommended for everyone.

Reference: http://projects.gnome.org/gnome-power-manager/

+ Print Queue Applet

A print job manager for CUPS to allow the user to be notified of printing and about new plugged printers to install.

Command: system-config-printer-applet

Status: wanted if you use any printers.

+ PulseAudio Sound System

The PulseAudio system offers a sound server for multiple sound sources and communicates with the underlying audio architecture (the most common in GNU/Linux are ALSA and OSS) to mix multiple audio with multiple audio cards and manage volume for each application.

Command: start-pulseaudio-x11

Status: recommended for everyone until you have not a sound card working.

+ PulseAudio Sound System KDE Routing

The PulseAudio starter for KDE. GNOME users don’t require this at all.

Command: start-pulseaudio-kde

Status: unwanted for every GNOME users.

+ Remote Desktop

The Vino server is a VNC server for incoming connections to the desktop, allowing others users to connect, view and operate in the desktop. This requires the Vino server was enabled in system preferences.

Command: /usr/lib/vino/vino-server

Status: unwanted until you’re absolutely sure what are doing.

+ Screensaver

The screen saver relies on this component to start automatically after the desired time indicated in the system preferences. This also will lock the screen when the computer is left alone for some minutes.

Command: gnome-screensaver

Status: recommended for everyone until you use another screen saver application.

+ Secret Storage Service

The GNOME Keyring components that contains the personal saved passwords for various applications (Evolution, WiFi) will be unlocked to grant the applications the right to read their saved passwords.

Command: /usr/bin/gnome-keyring-daemon --start --components=secrets

Status: generally wanted  if you save passwords into applications.

+ SSH Key Agent

A GNOME Keyring agent for SSH which will load your SSH keys from ~/.ssh in order to grant applications access to your SSH keys.
Command: /usr/bin/gnome-keyring-daemon --start --components=ssh

Status: unwanted if you don’t use SSH keys.

Reference: https://live.gnome.org/GnomeKeyring/Ssh

+ Volume Control

A GNOME classic applet for GNOME panel to let the user to arrange the volume of the audio.

Command: gnome-sound-applet

Status: unwanted until you use GNOME fallback session. Unneeded for GNOME Shell users.

----------------------------------------------------------------

<h3 id='lock-touchpad'> 2.2 锁定笔记本触控板 </h3>
在终端中输入：
```
sudo modprobe -r psmouse
```
即可锁定触控板。该命令仅在下次重启前有效。

----------------------------------------------------------------

<h3 id='lid'> 2.3 ubuntu中合上屏幕后无操作 </h3>

Modify the following line in file */etc/systemd/logind.conf*:
```
#HandleLidSwitch=suspend
```
to:
```
HandleLidSwitch=ignore
```
and restart the service by:
```
sudo systemctl restart systemd-logind.service
```

----------------------------------------------------------------

<h3 id='command'> 2.4 自定义命令 </h3>
Add the following contents to *~/.profile*:
```
export PATH="/media/uraplutonium/Workstation/Applications/upnbin:$PATH"
```
and put your own shell commands in that folder and make the file executable.
For example, the contents of */media/uraplutonium/Workstation/Applications/upnbin/upn-test* is as follow:
```
#!/bin/bash
# This script does nothing, but simply prints "Hello uraplutonium!" on the screen, and creates an empty file upntestfile in home folder.
# dependencies:
# [none]
# usage:
# upn-test

echo Hello uraplutonium!
touch ~/upntestfile
```

----------------------------------------------------------------

<h3 id='cent-ipv6'> 2.5 禁用CentOS7的IPv6和防火墙 </h3>
Upstream employee Daniel Walsh recommends not disabling the ipv6 module, as that can cause issues with SELinux and other components, but adding the following to */etc/sysctl.conf*:
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

To disable in the running system:
```
echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6
echo 1 > /proc/sys/net/ipv6/conf/default/disable_ipv6
```
or
```
sysctl -w net.ipv6.conf.all.disable_ipv6=1
sysctl -w net.ipv6.conf.default.disable_ipv6=1
```

To disable the firewall in the runing CentOS:
```
systemctl disable iptables.service
systemctl stop firewalld
```

----------------------------------------------------------------

<h3 id='sudoers'> 2.6 向sudo group中添加用户 </h3>
Add after the following line in file */etc/sudoers*:
```
root	ALL=(ALL) 	ALL
```
with:
```
uraplutonium	ALL=(ALL) 	ALL
```

----------------------------------------------------------------

<h3 id='apps'> 2.7 设置文件默认打开方式 </h3>

#### Local configuration 局部配置
Modify the file *~/.local/share/applications/mimeapps.list* or *~/.local/share/applications/mimeinfo.cache*. For example, we open all text and source code files by emacs24:  
修改文件*~/.local/share/applications/mimeapps.list* 或 *~/.local/share/applications/mimeinfo.cache*。以使用emacs24打开所有的文本和源代码文件为例：  
```
[Default Applications]
text/plain=emacs24.desktop
text/x-tex=emacs24.desktop
text/x-java=emacs24.desktop
text/x-csrc=emacs24.desktop
text/x-chdr=emacs24.desktop
text/x-c=emacs24.desktop
text/x-c++=emacs24.desktop
text/x-python=emacs24.desktop
text/xml=emacs24.desktop
text/x-emacs-lisp=emacs24.desktop
text/x-sql=emacs24.desktop
text/english=emacs24.desktop
text/x-makefile=emacs24.desktop
text/x-dtd=emacs24.desktop
text/mathml=emacs24.desktop
text/x-moc=emacs24.desktop
text/x-pascal=emacs24.desktop
text/x-tcl=emacs24.desktop
application/x-shellscript=emacs24.desktop
application/x-perl=emacs24.desktop
```

#### Global configuration 全局配置
Modify the following files:  
修改下列文件：  
*/etc/gnome/defaults.list*  
*/usr/share/applications/mimeinfo.cache*

----------------------------------------------------------------

<h3 id='backspace'> 2.8 启用nautilus上一级目录的BackSpace快捷键 </h3>
Modify the file *~/.config/nautilus/accels*:  
修改文件*~/.config/nautilus/accels*：  
```
(gtk_accel_path "<Actions>/ShellActions/Up" "BackSpace")
```

----------------------------------------------------------------

<h3 id='lenses'> 2.9 删除ubuntu unity桌面lenses </h3>
Just remove or rename the corresponding lens files in */usr/share/unity/lenses/*.  
删除或重命名/usr/share/unity/lenses/目录下对应的lens文件即可。

----------------------------------------------------------------

<h3 id='folders'> 2.10 更改Gnome桌面默认文件夹地址 </h3>
Modify the file */home/uraplutonium/.config/user-dirs.dirs*:  
修改文件*/home/uraplutonium/.config/user-dirs.dirs*：
```
# This file is written by xdg-user-dirs-update
# If you want to change or add directories, just edit the line you're
# interested in. All local changes will be retained on the next run
# Format is XDG_xxx_DIR="$HOME/yyy", where yyy is a shell-escaped
# homedir-relative path, or XDG_xxx_DIR="/yyy", where /yyy is an
# absolute path. No other format is supported.
# 
XDG_DESKTOP_DIR="$HOME"
XDG_DOWNLOAD_DIR="/media/uraplutonium/Workstation/Downloads"
XDG_TEMPLATES_DIR="/media/uraplutonium/Workstation/Recovery/Linux/Templates"
XDG_PUBLICSHARE_DIR="/media/uraplutonium/Workstation/Sync/Public"
XDG_DOCUMENTS_DIR="/media/uraplutonium/Workstation/Library"
XDG_MUSIC_DIR="/media/uraplutonium/Workstation/Music"
XDG_PICTURES_DIR="$HOME/Pictures"
```

----------------------------------------------------------------

<h3 id='ibus'> 2.11 gnome ibus设置 </h3>

开机自动启动ibus daemon:

Add the following codes in ~/.profile:

```
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
ibus-daemon -d -x
```

----------------------------------------------------------------

<h3 id='desktop'> 2.12 手动创建Gnome桌面快捷方式 </h3>

To manually create a desktop shortcut for a particular program or command, you can create a .desktop file using any text editor, and place it in either 
/usr/share/applications
or
~/.local/share/applications

```
[Desktop Entry]
Encoding=UTF-8
Version=1.0                                     # version of an app.
Name[en_US]=yEd                                 # name of an app.
GenericName=GUI Port Scanner                    # longer name of an app.
Exec=java -jar /opt/yed-3.11.1/yed.jar          # command used to launch an app.
Terminal=false                                  # whether an app requires to be run in a terminal.
Icon[en_US]=/opt/yed-3.11.1/icons/yicon32.png   # location of icon file.
Type=Application                                # type.
Categories=Application;Network;Security;        # categories in which this app should be listed.
Comment[en_US]=yEd Graph Editor                 # comment which appears as a tooltip.
```

----------------------------------------------------------------

<h3 id='gvfs'> 2.13 修复gvfs-metadata大量消耗内存 </h3>

```
pkill gvfsd-metadata
rm -rf ~/.local/share/gvfs-metadata
```
----------------------------------------------------------------

<h2 id='3'> 3. Linux commands & tools 命令与工具 </h2>

<h3 id='catsystem'> 3.1 查看系统信息 </h3>

#### Hardware 硬件

| Commands                         | Description                   |
| :------------------------------- | :---------------------------- |
| service kudzu start (or restart) | 用硬件检测程序kuduz探测新硬件 |
| cat /proc/cpuinfo                | 查看CPU信息                   |
| cat /proc/pci                    | 查看板卡信息                  |
| lspci                            | 查看PCI信息                   |
| cat /proc/meminfo                | 查看内存信息                  |
| cat /proc/bus/usb/devices        | 查看USB设备                   |
| cat /proc/bus/input/devices      | 查看键盘和鼠标                |
| fdisk & disk – l & df            | 查看系统硬盘信息和使用情况    |
| cat /proc/interrupts             | 查看各设备的中断请求(IRQ)     |
| dmesg /var/log/dmesg \| more     | 查看启动硬件检测信息日志      |

#### 系统

| Commands                | Description               |
| :---------------------- | :------------------------ |
| cat /etc/fedora-release | 查看fedora版本号          |
| uname -a                | 查看内核/操作系统/CPU信息 |
| head -n 1 /etc/issue    | 查看操作系统版本          |
| cat /proc/cpuinfo       | 查看CPU信息               |
| hostname                | 查看计算机名              |
| lspci -tv               | 列出所有PCI设备           |
| lsusb -tv               | 列出所有USB设备           |
| lsmod                   | 列出加载的内核模块        |
| env                     | 查看环境变量              |

#### 资源

| Commands                    | Description                    |
| :-------------------------- | :----------------------------- |
| free -m                     | 查看内存使用量和交换区使用量   |
| df -h                       | 查看各分区使用情况             |
| du -sh                      | 查看指定目录的大小             |
| grep MemTotal /proc/meminfo | 查看内存总量                   |
| grep MemFree /proc/meminfo  | 查看空闲内存量                 |
| uptime                      | 查看系统运行时间、用户数、负载 |
| cat /proc/loadavg           | 查看系统负载                   |

#### 磁盘和分区

| Commands           | Description                   |
| :----------------- | :---------------------------- |
| mount \| column -t | 查看挂接的分区状态            |
| fdisk -l           | 查看所有分区                  |
| swapon -s          | 查看所有交换分区              |
| hdparm -i /dev/hda | 查看磁盘参数(仅适用于IDE设备) |
| dmesg \| grep IDE  | 查看启动时IDE设备检测状况     |

#### 网络

| Commands      | Description            |
| :------------ | :--------------------- |
| ifconfig      | 查看所有网络接口的属性 |
| iptables -L   | 查看防火墙设置         |
| route -n      | 查看路由表             |
| netstat -lntp | 查看所有监听端口       |
| netstat -antp | 查看所有已经建立的连接 |
| netstat -s    | 查看网络统计信息       |

#### 进程

| Commands | Description      |
| :------- | :--------------- |
| ps -ef   | 查看所有进程     |
| top      | 实时显示进程状态 |

#### 用户

| Commands                | Description                       |
| :---------------------- | :-------------------------------- |
| w                       | 查看活动用户                      |
| id                      | 查看指定用户信息                  |
| last                    | 查看用户登录日志                  |
| cut -d: -f1 /etc/passwd | 查看系统所有用户                  |
| cut -d: -f1 /etc/group  | 查看系统所有组                    |
| crontab -l              | 查看当前用户的计划任务            |
| useradd centospub       | 建立用户名为 centospub 的一般用户 |
| passwd centospub        | 为用户 centospub 设置密码         |
| userdel -r centospub    | 删除用户名为 centospub 的一般用户 |

#### 服务

| Commands                   | Description            |
| :------------------------- | :--------------------- |
| chkconfig –list            | 列出所有系统服务       |
| chkconfig –list \| grep on | 列出所有启动的系统服务 |
| service sshd start         | 启动服务               |
| service sshd stop          | 停止服务               |
| service sshd restart       | 重启服务               |

#### 程序

| Commands | Description          |
| :------- | :------------------- |
| rpm -qa  | 查看所有安装的软件包 |

#### Linux查询目录使用空间

| Commands       | Description                                                                           |
| :------------- | :------------------------------------------------------------------------------------ |
| du -sh dirname | 查看目录的使用空间                                                                    |
| du -s dirname  | 仅显示总计                                                                            |
| du -h dirname  | 以k、m、g为单位，提高信息的可读性。 kb、mb、gb是以1024为换算单位， -h以1000为换算单位 |
| du -a dirname  | 显示全部目录和其次目录下的每个档案所占的磁碟空间                                      |
| du -b dirname  | 大小用bytes来表示(预设值为k bytes)                                                    |
| du -c dirname  | 最后再加上总计(预设值)                                                                |
| du -l dirname  | 计算所有档案大小                                                                      |
| du -x dirname  | 只计算同属同一个档案系统的档案                                                        |
| du -L dirname  | 计算所有的档案大小                                                                    |

----------------------------------------------------------------

<h3 id='crontab'> 3.2 crontab计划任务 </h3>
crontab可以定时、周期性地执行命令或脚本。
以“每周一凌晨3点执行备份文件的脚本upn-backup.sh”为例：
#### 首先，安装crontab软件：

```
sudo apt-get install corntab
```

#### 然后，创建crontab文件（~/crontab-file）：

```
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command

0 3	* * 1	/home/uraplutonium/upn-backup.sh
```

如默认crontab文件的注释所说，每一行的前5个数字分别代表执行命令的分钟（0-59）、小时（0-23）、日期（1-31）、月份（1-12）和星期（1-7）。星号“*”代表不作限制。“0 3	* * 1”代表每个月、每个星期一的03点00分执行命令。

#### 最后，在终端中输入下列语句来加载crontab文件：

```
crontab ~/crontab-file
```

要查看是否设置成功，可以执行下列命令来查看当前所有的计划任务：
```
crontab -l
```

----------------------------------------------------------------

<h3 id='rsync'> 3.3 rsync文件同步 </h3>

```bash
#!/bin/bash
rsync -r -u -p -o -g -t --delete --progress --exclude=lost+found --exclude=.Trash-1000 /media/uraplutonium/Workstation/ /media/uraplutonium/Resource/
```

----------------------------------------------------------------

<h3 id='proxy'> 3.4 通过代理服务器使用yum和dnf </h3>

Add the following lines to the files:
/etc/yum.conf
or
/etc/dnf/dnf.conf

```
proxy_username=your_user_name
proxy_password=your_password
proxy=http://ip_address:port
```

----------------------------------------------------------------

<h3 id='maven'> 3.5 maven local repository </h3>

edit the /apache-maven-3.3.3/conf/setting.xml

```xml
<!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  -->
  <localRepository>/media/uraplutonium/Workstation/Applications/maven-repo</localRepository>
```
  
----------------------------------------------------------------

<h3 id='gnome-notify'> 3.6 向Gnome桌面发送通知 </h3>

```
notify-send -t 3000 "the message"
```

----------------------------------------------------------------

<h3 id='ssh-pwd'> 3.7 ssh免密码登录 </h3>

workstation为本地主机，outpost为远程主机
在workstation上生成无密码的私钥和公钥：
```
ssh-keygen -t rsa
ssh -p 28366 uraplutonium@outpost "mkdir .ssh; chmod 0700 .ssh"
scp -P 28366 /home/uraplutonium/.ssh/id_rsa.pub uraplutonium@outpost:.ssh/id_rsa.pub
```

在outpost上执行命令：
```
touch /home/uraplutonium/.ssh/authorized_keys
chmod 600 /home/uraplutonium/.ssh/authorized_keys
cat /home/uraplutonium/.ssh/id_rsa.pub >> /home/uraplutonium/.ssh/authorized_keys
```

在workstation上即可以使用ssh免密码登录outpost：
```
ssh -p 28366 uraplutonium@outpost
```

----------------------------------------------------------------

<h3 id='ssh-tunnel'> 3.8 ssh反向隧道 </h3>

将本机的ssh22号端口映射到outpost VPS的23333端口：
```
ssh -NfR 23333:127.0.0.1:22 uraplutonium@outpost -p 28366
```

登录到outpost VPS:
```
ssh uraplutonium@outpost -p 28366
```

在outpost VPS上登录到workstation:
```
ssh uraplutonium@localhost -p 23333
```

在outpost VPS上可以使用以下命令查看连接:
```
netstat -tnl | grep 127.0.0.1
```

----------------------------------------------------------------

<h3 id='samba'> 3.9 samba共享文件夹设置 </h3>

Add the following lines to /etc/samba/smb.conf:
```
[Public]
   comment = Public
   path = /home/uraplutonium/YandexDisk/Public
   browseable = yes
   read only = no
   guest ok = yes
   writable = yes
   available = yes
   public = yes
   create mask = 0755
   directory mask = 0755
```
   
Run:
```
sudo /etc/init.d/samba reload
```

----------------------------------------------------------------

<h3 id='php5'> 3.10 apache2和php5网站 </h3>

#### 1. install apache2 php5 libapache2-mod-php5

#### 2. make the home folder of the site:

```
mkdir /media/uraplutonium/Workstation/Workspace/sites/
```

#### 3. append the following lines to /etc/apache2/sites-available/000-default.conf

```
Alias / "/media/uraplutonium/Workstation/Workspace/sites/"
<Directory "/media/uraplutonium/Workstation/Workspace/sites/">
    Order allow,deny
    Allow from all
    # New directive needed in Apache 2.4.3: 
    Require all granted
	</Directory>
```
	
#### 4. restart apache2

```
sudo /etc/init.d/apache2 restart
```

#### 5. sites in /media/uraplutonium/Workstation/Workspace/sites/subfolder/site.php will be available at http://127.0.0.1/subfolder/site/php

The default page http://127.0.0.1/ is directed to /media/uraplutonium/Workstation/Workspace/sites/index.html

----------------------------------------------------------------

<h3 id='ffmpeg'> 3.11 将视频转换为mp4格式 </h3>

```
ffmpeg -i $1 \
	-c:v libx264 -preset veryslow -crf 22 \
	-c:a libmp3lame -qscale:a 2 -ac 2 -ar 44100 \
	video.mp4z
```

----------------------------------------------------------------
