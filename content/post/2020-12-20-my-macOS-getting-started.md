---
title:       "我的 macOS 设置"
summary:     "记录我的 macOS 的特殊设置，包括但不限于一些工具的国内镜像设置。"
date:        2020-12-19
author:      "Luiz Yao"
tags:        ["macOS"]
categories:  ["笔记"]
draft: false
toc: true
---

> 目前 macOS 的版本是：macOS Big Sur 11.1

## HomeBrew

目前使用的国内源是[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/)。

### 使用 `HomeBrew` 镜像

```zsh
# brew 程序本身
$ git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

$ git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

$ git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

$ git -C "$(brew --repo homebrew/cask-fonts)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask-fonts.git

# 更换后测试工作是否正常
$ brew update
```

### 使用 `Homebrew-bottles` 镜像

`Homebrew-bottles` 提供 `Homebrew` 二进制预编译包的镜像，用于常见工具的下载。

```zsh
$ echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zprofile
$ source ~/.zprofile
```

## ~~npm~~

### 国内镜像

使用淘宝的 `cnpm` 取代官方的 `npm`：

```zsh
#  全局安装 cnpm
$ npm install -g cnpm --registry=https://registry.npm.taobao.org

# 使用 cnpm 安装其它工具，例如：gitmoji-cli
$ cnpm i -g gitmoji-cli
```

## 终端

### 配置代理

> 参考：<https://github.com/mrdulin/blog/issues/18>

在 `~/.zshrc` 文件中添加如下配置：

```zsh
alias proxy='export all_proxy=socks5://127.0.0.1:1086'
alias unproxy='unset all_proxy'
```

重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

使用代理的情况：

```zsh
$ proxy

$ curl cip.cc
IP	: 149.28.78.133
地址	: 美国  加利福尼亚州  洛杉矶
运营商	: choopa.com

数据二	: 美国 | 加利福尼亚州洛杉矶Choopa数据中心

数据三	: 美国加利福尼亚

URL	: http://www.cip.cc/149.28.78.133
```

关闭代理的情况：

```zsh
$ unproxy

$ curl cip.cc
IP	: 121.225.27.7
地址	: 中国  江苏  南京
运营商	: 电信

数据二	: 江苏省南京市 | 电信

数据三	: 中国江苏南京 | 电信

URL	: http://www.cip.cc/121.225.27.7
```

## GitHub

### SSH 代理

修改 `~/.ssh/config` 文件：

```conf
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa
    ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
```

### 修改 hosts 解决访问慢的问题

> 参考：<https://github.com/521xueweihan/GitHub520>

安装 **switchhosts**：

```zsh
$ brew install switchhosts
```

配置如下：

![switchhosts 配置](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-22_05-46-55.png)

- URL：<https://gitee.com/xueweihan/codes/6g793pm2k1hacwfbyesl464/raw?blob_name=GitHub520.yml>

