---
title:       "在树莓派上安装 Docker 和 Docker Compose"
subtitle:    ""
summary:     "简单介绍如何在树莓派上安装 Docker 和 Docker Compose。"
date:        2020-12-05
author:      "Luiz Yao"
tags:        ["Raspberry Pi", "Docker"]
categories:  ["Linux"]
draft: false
---

[Docker](https://www.docker.com/) 是一款流行的 Linux 容器解决方案。本文介绍一些在树莓派上使用 Docker 的实践。

> 本文使用的是树莓派版本是 `Raspbian GNU/Linux 10 (buster)`。

# 安装 Docker

> 此部分主要参考官方文档中的 [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)。

## 卸载旧版本

```bash
pi@raspberrypi:~ $ sudo apt remove docker docker-engine docker.io containerd runc
```

## 使用官方脚本安装

```bash
pi@raspberrypi:~ $ curl -fsSL https://get.docker.com -o get-docker.sh
pi@raspberrypi:~ $ sudo sh get-docker.sh
```

我们将当前用户 `pi` 加入到 `docker` 用户组中，这样就不需要使用 `sudo` 赋权了。

```bash
pi@raspberrypi:~ $ sudo usermod -aG docker pi
```

这个可能需要我们重新登录才能生效。

## 卸载

```bash
pi@raspberrypi:~ $ sudo apt purge docker-ce docker-ce-cli containerd.io
pi@raspberrypi:~ $ sudo rm -rf /var/lib/docker
pi@raspberrypi:~ $ sudo rm -rf /var/lib/containerd
```

# 安装 Docker Compose

Docker Compose 方便我们同时管理多个容器。

在树莓派上我们可以通过 `pip` 来安装，但是这要求 `Python >= 3.6`。

首先，安装 `Python3`：

```bash
pi@raspberrypi: ~ $ sudo apt install python3
```

然后，安装 `docker-compose`：

```bash
pi@raspberrypi: ~ $ pipx install docker-compose
```

---

**NOTE**

[pipx](https://github.com/pipxproject/pipx) 可以让我们在隔离的环境中使用 Python 开发的命令行工具，类似于 macOS 上面的 `brew`；

只需要以下几个命令就可以安装它：

```bash
pi@raspberrypi:~ $ python3 -m pip install --user pipx
pi@raspberrypi:~ $ python3 -m pipx ensurepath
pi@raspberrypi:~ $ pipx completions
```

---

# 国内镜像加速

修改或创建 `/etc/docker/daemon.json` 文件，加入以下内容：

```json
{
    "registry-mirrors": [
    	"https://hub-mirror.c.163.com",
    	"https://mirror.baidubce.com"
	]
}
```

再重启 Docker 服务：

```bash
pi@raspberrypi:~ $ sudo systemctl daemon-reload
pi@raspberrypi:~ $ sudo systemctl restart docker
```

