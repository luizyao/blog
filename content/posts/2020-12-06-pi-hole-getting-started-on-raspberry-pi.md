---
title:       "pi-hole：搭建自己的 Ad Blocker"
subtitle:    ""
summary:     "pi-hole 可以帮助我们在自己的 Linux 设备上搭建一个 AD Blocker。本文就是介绍它在树莓派上的一些实践。"
date:        2020-12-06
author:      "Luiz Yao"
image:       ""
tags:        ["Raspberry Pi", "pi-hole"]
categories:  ["Github 漫游"]
draft: false
---

最近，在 Github 上发现一个有趣的项目 —— [pi-hole](https://github.com/pi-hole/pi-hole)，它可以帮助我们在网络侧“狙击”广告，提供更好的上网体验。

本文介绍如何使用它在我们的树莓派上搭建一个 Ad Blocker。

> 如果你不知道树莓派是什么，可以参考我的这篇文章：[树莓派入门](/2020-11-28-raspberry-pi-getting-started)。

pi-hole 之所以能在网络侧过滤广告，使用的是一种叫做 [DNS sinkhole](https://en.wikipedia.org/wiki/DNS_sinkhole) 的技术，它可以作为一个 DNS 服务器，但是提供的是错误的 DNS 解析。

# 安装

pi-hole 可以做一个 [Docker 容器](https://github.com/pi-hole/docker-pi-hole)来提供服务。

>如果你不知道怎么在树莓派上安装 Docker，可以参考我的这篇文章：[在树莓派上安装 Docker 和 Docker Compose](/2020-12-05-docker-installation-on-raspberry-pi)

首先，我们需要编写 `docker-compose.yml` 文件（在官方给的示例上修改）：

```yaml
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      ADMIN_EMAIL: "yaomeng614@gmail.com"
      TZ: "Asia/Shanghai"
      # WEBPASSWORD: "set a secure password here or it will be random"
      DNS1: "114.114.114.114"
      DNS2: "114.114.115.115"
    # Volumes store your data between container upgrades
    volumes:
      - "./etc-pihole:/etc/pihole"
      - "./etc-dnsmasq.d:/etc/dnsmasq.d"
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

- `WEBPASSWORD`：WEB 管理界面的登录密码。默认是一个随机字符串，可以使用 `docker logs pihole | grep random` 命令查看当前的随机密码。

- `DNS1` 和 `DNS2`：默认是`8.8.8.8`和`8.8.4.4`，这里使用国内 DNS 服务器`114.114.114.114`和`114.114.115.115`。

然后，运行 pi-hole 服务：

```bash
pi@raspberrypi: $ docker-compose up -d
```

查看结果：

```bash
pi@raspberrypi: $ docker ps
CONTAINER ID        IMAGE                  COMMAND             CREATED             STATUS                            PORTS                                                                                                  NAMES
2d21aab833df        pihole/pihole:latest   "/s6-init"          11 seconds ago      Up 6 seconds (health: starting)   0.0.0.0:53->53/udp, 0.0.0.0:53->53/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:67->67/udp   pihole
```

# WEB 管理界面

现在我们可以访问 pi-hole 的管理页面，这里的地址是 <http://raspberrypi.local/admin/>：

![pi-hole 管理页面的首页](https://gitee.com/luizyao/pictures/raw/master/img/pi-hole-admin-web-homepage.png)

然后，通过 `docker logs pihole | grep random` 可以获取生成的随机密码，登录后就能解锁更多功能：

![pi-hole 登录解锁更多内容](https://gitee.com/luizyao/pictures/raw/master/img/pi-hole-admin-web-after-login.png)

# 使用

## 单台设备使用

以我的 mac  为例，只需要为当前连接的网络的 DNS 服务器指定为 pi-hole 的 IP 地址：

1. 打开 **Apple > System Preferences > Network**；

2. 选择当前连接的网络；
3. 点击 **Advanced**；
4. 选择 **DNS**；
5. 添加你的树莓派的 IP 地址；
6. 点击 **OK > Apply**；

# 更新 Adlists

pi-hole 根据域名过滤不想要的内容，其本身内置的 Adlists 可能不太符合国内的情形。我目前使用的是 [anti-AD](https://github.com/privacy-protection-tools/anti-AD) 提供的 Adlists，它的使用也非常简单：

打开 pi-hole 的管理页面，点击 **Group Management > Adlists**，添加 Adlists 地址：<https://anti-ad.net/domains.txt>。

然后，进入 **Tools > Update Gravity**，点击 **Update** 按钮，稍等片刻即可。

这时我们发现黑名单中的域名明显增多了：

![更新 Adlists](https://gitee.com/luizyao/pictures/raw/master/img/pi-hole-update-adlists.png)

# 效果

现在我们来看看效果吧。

使用 pi-hole 之前：

![使用 pi-hole 之前](https://gitee.com/luizyao/pictures/raw/master/img/pi-hole-disable.png)

使用 pi-hole 之后：

![使用 pi-hole 之后](https://gitee.com/luizyao/pictures/raw/master/img/pi-hole-enable.png)

相对而言，清爽了许多。

pi-hole 并不能去除所有的广告，例如腾讯视频中开头的广告（虽然这是我尝试这个项目的初衷）。但是我觉得它能做到的是节省带宽，过滤掉无意义的请求报文，提供更快速的上网体验。所以，还是值得一试的。

