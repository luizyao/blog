---
title:       "使用 starship 自定义树莓派的终端"
subtitle:    ""
summary:     "starship 是使用 Rust  编写的一个开源工具，它可以自定义各种 Shell（Bash、Fish、Zsh 等） 的终端提示符。本文介绍它在树莓派上的实践。"
date:        2020-12-05
author:      "Luiz Yao"
image:       ""
tags:        ["Raspberry Pi", "starship"]
categories:  ["Github 漫游"]
draft: true
---

[starship](https://github.com/starship/starship) 是使用 Rust  编写的一个开源工具，它可以自定义各种 Shell（Bash、Fish、Zsh 等） 的终端提示符。

现在，让我们在树莓派上试一试吧！先来查看一下当前使用的 shell：

```bash
pi@raspberrypi:~ $ echo $SHELL
/bin/bash
```

这里可以看到使用的是 `bash`。

# 安装

首先，我们需要先安装一种 [Nerd Font](https://github.com/ryanoasis/nerd-fonts) 字体，这里选择 [Fira Code Nerd Font](https://github.com/tonsky/FiraCode)。

```bash
pi@raspberrypi:~ $ sudo apt install fonts-firacode
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following package was automatically installed and is no longer required:
  lxplug-volume
Use 'sudo apt autoremove' to remove it.
The following NEW packages will be installed:
  fonts-firacode
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 890 kB of archives.
After this operation, 1,398 kB of additional disk space will be used.
Get:1 http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian buster/main armhf fonts-firacode all 1.206+dfsg1-4 [890 kB]
Fetched 890 kB in 1s (1,192 kB/s)
Selecting previously unselected package fonts-firacode.
(Reading database ... 101413 files and directories currently installed.)
Preparing to unpack .../fonts-firacode_1.206+dfsg1-4_all.deb ...
Unpacking fonts-firacode (1.206+dfsg1-4) ...
Setting up fonts-firacode (1.206+dfsg1-4) ...
Processing triggers for fontconfig (2.13.1-2) ...
```

然后，开始安装 starship：

```bash
pi@raspberrypi:~ $ curl -fsSL https://starship.rs/install.sh | bash
```

---

**注意**

目前还不支持树莓派：

```bash
pi@raspberrypi:~ $ curl -fsSL https://starship.rs/install.sh | bash

  Configuration
> Bin directory: /usr/local/bin
> Platform:      unknown-linux-musl
> Arch:          armv7l

> Tarball URL: https://github.com/starship/starship/releases/latest/download/starship-armv7l-unknown-linux-musl.tar.gz
? Install Starship latest to /usr/local/bin? [y/N] y
! Escalated permissions are required to install to /usr/local/bin

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for pi:
> Installing Starship as root, please wait…

x Command failed (exit code 22): curl --silent --fail --location https://github.com/starship/starship/releases/latest/download/starship-armv7l-unknown-linux-musl.tar.gz

> This is likely due to Starship not yet supporting your configuration.
> If you would like to see a build for your configuration,
> please create an issue requesting a build for armv7l-unknown-linux-musl:
> https://github.com/starship/starship/issues/new/\n

gzip: stdin: unexpected end of file
tar: Child returned status 1
tar: Error is not recoverable: exiting now
```

但是已经有一个 [Rasbperry 3 (armv7l-unknown-linux-musl)](https://github.com/starship/starship/issues/1736) 在记录。

**保持关注中～～～**

---

