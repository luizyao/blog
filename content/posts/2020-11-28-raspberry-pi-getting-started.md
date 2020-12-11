---

title:       "树莓派入门"
subtitle:    ""
summary:     "简单的介绍树莓派的背景，着重于树莓派的设置，包括安装操作系统、设置无线上网、系统备份等。"
date:        2020-11-28
lastmod: 	 2020-12-05
author:      "Luiz Yao"
tags:        ["Raspberry Pi"]
categories:  ["Linux"]
draft: false

featuredImage: "https://gitee.com/luizyao/pictures/raw/master/img/pi-labelled-names.png"
featuredImagePreview: false
---

## 背景

[树莓派（Raspberry Pi）](https://www.raspberrypi.org/)是基于 Linux 的[单片机](https://zh.wikipedia.org/wiki/单片机)电脑，由英国的树莓派基金会开发，目的是以低价硬件及自由软件促进学校的基本计算机科学教育。

树莓派使用博通（Broadcom）的 ARM 架构处理器，内存在 **2GB** 和 **8GB** 之间，主要使用 **SD 卡**作为存储媒体，配备 USB、HDMI 等接口，支持有线、无线、蓝牙网络连接方式，并且可以使用多种操作系统。

## 安装操作系统

[Raspberry Pi OS](https://www.raspberrypi.org/software/) 是官方支持操作系统，除此之外还有一些第三方的操作系统，例如：[Ubuntu MATE](https://ubuntu-mate.org/)、[Ubuntu Core](https://ubuntu.com/core) 等。

这里我们安装的是 Raspberry Pi OS。

### 使用 Raspberry Pi Imager 安装操作系统

Raspberry Pi Imager 可以帮助我们快速地将 Raspberry Pi OS 或者其它操作系统安装到一个 microSD 卡上面。

首先，在官网下载对应操作系统的安装包，支持 macOS、Windows 和 Ubuntu for x86。

本文以 macOS 为例。

1. 选择想要安装的操作系统：

    因为直接下载速度会比较慢，所以可以通过国内的镜像源先把下载系统文件。

    这里使用的是清华源：<https://mirrors.tuna.tsinghua.edu.cn/raspberry-pi-os-images/>

    目前最新的版本是：<https://mirrors.tuna.tsinghua.edu.cn/raspberry-pi-os-images/raspios_armhf/images/raspios_armhf-2020-08-24/2020-08-20-raspios-buster-armhf.zip>

    最后选择刚才下载的文件。

    ![选择想要安装的操作系统](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-imager-usage-select-os.png)

2. 选择 SD 卡：

    ![选择 SD 卡](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-imager-usage-select-sd-card.png)

3. 将系统写入到 SD 卡：

    ![将系统写入到 SD 卡](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-imager-usage-write.png)

    ---

    **注意**

    这会先清除 SD 卡上的已有内容，选择 **YES** 开始写入。

    ![清除 SD 卡上的已有内容](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-imager-usage-erase.png)

    ---

4. 等待完成：

    ![完成](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-imager-usage-finished.png)

## 设置

至此，我们可以将 SD 卡插入树莓派中，接通电源，然后外接显示器、键盘、鼠标就可以正常使用了。如果我们没有这些设备，可能就要多做一些工作了。

### 设置无线网络

最好能让树莓派在启动时自动连接我们的无线路由器。

在 SD 卡的`boot` 文件夹下新建一个`wpa_supplicant.conf`文件，写入以下内容：

```text
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<ISO 3166-1标准定义的国家代码，中国是CN>

network={
    ssid="<SSID 名称>"
 	psk="<SSID 密码>"
}
```

>  更多关于`wpa_supplicant.conf`的细节可以参考：<https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md>

### 使能 SSH

自从2016年11月份的更新之后，Raspberry Pi OS 默认关闭了 SSH 的功能。

我们在设置无线网络的同时，可以很方便的同步使能 SSH 的功能。在`boot`文件夹下新建一个`ssh`的空文件，树莓派在启动时，会自动使能 SSH，并删除这个文件。

## 远程登录

现在我们就可以将 SD 卡插入树莓派中，并接通电源。

### 探测 IP 地址

树莓派通电后，我们需要知道它的 IP 地址。最简单的方法，就是查看你的路由器，设备名称是`raspberrypi`就是我们的树莓派了。

除此之外，如果你的电脑支持 [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) 功能，你还可以通过`<hostname>.local`地址访问你的树莓派，默认的`hostname`是`raspberrypi`，所以我们可以通过 `ping raspberrypi.local`获取它的 IP 地址。

```bash
$ ping raspberrypi.local
PING raspberrypi.local (192.168.31.142): 56 data bytes
64 bytes from 192.168.31.142: icmp_seq=0 ttl=64 time=5.497 ms
```

> 更多探测树莓派 IP 地址的方法可以参考：<https://www.raspberrypi.org/documentation/remote-access/ip-address.md>

### SSH 登录

默认的用户名是`pi`，密码是`raspberry`。

```bash
$ ssh pi@raspberrypi.local
The authenticity of host 'raspberrypi.local (fe80::fae4:4a4f:de4d:2333%en0)' can't be established.
ECDSA key fingerprint is SHA256:l4AODUWJN68FjzD6AguCID2eO0BoDIKvHObIakalIRI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'raspberrypi.local,fe80::fae4:4a4f:de4d:2333%en0' (ECDSA) to the list of known hosts.
pi@raspberrypi.local's password:
Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Aug 20 11:54:38 2020

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ whoami
pi
```

查看当前的系统的发行信息：

```bash
pi@raspberrypi:~ $ sudo lsb_release -a
No LSB modules are available.
Distributor ID:	Raspbian
Description:	Raspbian GNU/Linux 10 (buster)
Release:	10
Codename:	buster
```

### 更新系统

基于安全的考虑，我们最好更新一下系统，最简单的方法就是使用 APT 命令行工具。

#### 使用国内的 APT 源

为了获取更好的使用体验，我们可以将 [APT (Advanced Packaging Tool)](https://www.raspberrypi.org/documentation/linux/software/apt.md)  替换成国内清华源：

修改`/etc/apt/sources.list`文件，替换成如下内容：

```text
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main contrib rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib rpi
```

修改`/etc/apt/sources.list.d/raspi.list`文件，替换成如下内容：

```text
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main
```

更新软件源列表：

```bash
pi@raspberrypi:~ $ sudo apt-get update
```

---

**注意**

- 更多细节可以参考：<https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/>。
- 建议修改文件之前，先备份一下。

#### 更新 Raspberry Pi OS 系统

首先，更新系统的软件包列表：

```bash
pi@raspberrypi:~ $ sudo apt update
```

然后，将所有已安装的软件包升级到最新版本：

 ```bash
pi@raspberrypi:~ $ sudo apt full-upgrade
 ```

### 优化

我们还有一些优化项，可以让我们的树莓派使用起来更方便和安全。

#### 使能基于密钥的 SSH 认证方式

基于密钥的认证方式比基于密码的认证方式更加的安全。

首先，产生一个新的密钥对：

```bash
$ ssh-keygen -t ecdsa
```

然后，将公共密钥上传到树莓派的`~/.ssh/authorized_keys`文件中：

```bash
$ ssh-copy-id pi@raspberrypi.local
```

 这时候我们既可以使用基于密钥的登录方式，也可以使用基于密码的登录方式。

```bash
$ ssh pi@raspberrypi.local
Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Nov 29 10:02:25 2020 from fe80::32:2e96:65:d0da%wlan0

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $
```

#### 禁用基于密码的认证方式

现在我们仍然可以使用基于密码的认证，这仍然是有风险的，下面我们就禁用基于密码的认证。

首先，修改`/etc/ssh/sshd_config`文件，将其中的`PasswordAuthentication`改成`no`。

然后，重启 SSH 服务：

```bash
pi@raspberrypi:~ $ sudo systemctl restart sshd
```

这时候，我们就只能使用密钥登录：

```bash
$ ssh pi@raspberrypi.local
Linux raspberrypi 5.4.51-v7l+ #1333 SMP Mon Aug 10 16:51:40 BST 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Nov 29 11:26:04 2020 from fe80::32:2e96:65:d0da%wlan0
pi@raspberrypi:~ $
```

可以看到这里明显少了以下两行修改默认密码的提示信息，因为已经无法使用密码登录了。

```bash
SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.
```

### VNC (Virtual Network Computing)  远程桌面共享

即使我们没有多余的显示器、鼠标和键盘，我们也可以通过 [VNC](https://www.realvnc.com/en/) 远程共享我们的树莓派桌面。

首先，我们需要开启树莓派的 VNC 服务：

```bash
pi@raspberrypi:~ $ sudo raspi-config
```

在弹出来的图形界面选择 **Interfacing Options**，再将 **VNC** 选项置为 **Yes**，保存退出。

然后，我们需要在本机安装 VNC 客户端，下载地址为：<https://www.realvnc.com/en/connect/download/viewer/>。

现在，我们就可以使用树莓派的桌面服务了。默认的用户名是 `pi`，密码是 `raspberry`。

![树莓派桌面](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-vnc-desktop.png)

---

**注意**

你可能会遇到`Cannot currently show the desktop`的问题：

![Cannot currently show the desktop](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-vnc-cannot-show-desktop.png)

通常只需要调整树莓派的分辨率就可以解决这个问题：

命令行执行：

```bash
pi@raspberrypi:~ $ sudo raspi-config
```

选择 **Display Options** -> **Resolution**，最后选择分辨率，保存退出，等待设备重启。

我使用的分辨率是 **1920 * 1080**。

---

## 系统备份

规律地备份系统是一个良好的习惯。树莓派系统的备份分为备份`home`文件夹和备份整个 SD 卡。

### 备份 `home` 文件夹

[Déjà Dup](https://wiki.gnome.org/Apps/DejaDup) 软件可以帮助我们轻松地完成这个任务，它提供了可视化的配置窗口和自动定时备份的功能，我们可以很容易的在树莓派上安装它。

```bash
pi@raspberrypi:~ $ sudo apt-get update && sudo apt-get upgrade
pi@raspberrypi:~ $ sudo apt-get install deja-dup
```

安装完成后，我们在 **Accessories** -> **Backups** 中打开它：

![打开 Deja Dup](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-deja-dup-open.png)

配置界面如下：

![配置 Deja Dup](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-deja-dup-config.png)

它的配置非常直观，我们就不多做介绍了。

---

**注意**

这里我将备份文件夹选择我的硬盘，但是因为硬盘的格式是 `exFAT`，在树莓派上还需要安装额外的库才能识别：

```bash
pi@raspberrypi:~ $ sudo apt-get update && sudo apt-get upgrade
pi@raspberrypi:~ $ sudo apt-get install exfat-fuse
pi@raspberrypi:~ $ sudo apt-get install exfat-utils
```

---

### 备份整个 SD 卡

#### 使用 SD Card Copier 备份

它是树莓派自带的软件，可以将完整的系统拷贝到另一张 SD 卡上，这需要将新的 SD 卡接入到树莓派的 USB 插口上。

它的位置是 **Accessories** -> **SD Card Copier** ，使用起来也非常的直观。/Users/yaomeng/Private/projects/blog/static

![SD Card Copier](https://gitee.com/luizyao/pictures/raw/master/img/raspberry-pi-sd-card-copier.png)

---

**注意**

它会先擦除新的 SD 卡上所有的内容，并且要求新的 SD 卡的空间大于目前的卡空间。

---

#### 使用 `dd` 命令备份

这需要我们将 SD 卡从树莓派中拔出来，插到我们的电脑上。

以 macOS 为例（不同的系统命令可能不一样）：

首先，使用`df -f` 命令查看挂载的 SD 卡，在 macOS 上一般是 `/dev/disk2s1`。

然后，开始备份，时间较长，请耐心等待：

```bash
$ sudo dd bs=4m if=/dev/rdisk2 | gzip > PiOS.img.gz
```

这里我们使用 gzip 工具来压缩生成的 image 文件。

如果我们想要恢复备份，只需要这样：

```bash
$ gunzip --stdout PiOS.img.gz | sudo dd bs=4m of=/dev/rdisk2
```

我们可以再另开一个 shell，输入命令：`sudo killall -29 dd`，可以查看当前 `dd` 的进度：

```bash
0+478209 records in
0+478208 records out
31339839488 bytes transferred in 2318.707881 secs (13516079 bytes/sec)
```

---

**注意**

有时候我们会遇到 `Resource busy` 的问题：

```bash
$ gunzip --stdout PiOS.img.gz | sudo dd bs=4m of=/dev/rdisk2
dd: /dev/rdisk2: Resource busy
```

这是因为 SD 卡的分区已经挂载到系统中，所以我们要先卸载 SD 卡的分区。

首先，通过 `df -h ` 查看已挂载的分区：

```bash
$ df -h
Filesystem      Size   Used  Avail Capacity iused               ifree %iused  Mounted on
...
/dev/disk2s1    30Gi  1.9Mi   30Gi     1%       0                   0  100%   /Volumes/SDCARD
```

这里是 `/dev/disk2s1`。

然后，卸载分区：

```bash
$ sudo diskutil unmount /dev/disk2s1
Volume SDCARD on disk2s1 unmounted
```

再通过 `df -h` 命令确保分区已卸载。

现在，我们可以重新开始执行命令：

```bash
$ gunzip --stdout PiOS.img.gz | sudo dd bs=4m of=/dev/rdisk2
```

最后，命令执行结束后，SD 卡会重新挂载到系统中，并且名字是 `boot`：

```bash
$ df -h
Filesystem      Size   Used  Avail Capacity iused               ifree %iused  Mounted on
...
/dev/disk2s1   252Mi   55Mi  197Mi    22%       0                   0  100%   /Volumes/boot
```

> 参考：<https://raspberrypi.stackexchange.com/questions/9217/resource-busy-error-when-using-dd-to-copy-disk-img-to-sd-card>

---

> 更多的备份方法请参考：<https://www.raspberrypi.org/documentation/linux/filesystem/backup.md>

