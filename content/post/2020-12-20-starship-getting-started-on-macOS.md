---
title:       	"我的 iTerm2 终端方案"
summary:     	"使用 iTerm2 + zsh + starship 打造自己的 macOS 终端方案。"
date:        	2020-12-20
author:      	"Luiz Yao"
tags:        	["macOS", "starship", "iTerm2"]
categories:  	["笔记"]
draft: 			true
toc: 			true
featureImage:	"https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-21_19-27-05.png"
---

## iTerm2

### 安装

[iTerm2](https://iterm2.com/) 是 macOS 的终端模拟器，它有更高的可玩性，颜值也更高。我们可以直接使用 `homebrew` 安装它。

```zsh
$ brew install iterm2
```

### 修改字体

这里我选择的是 [Fira Code Nerd Font](https://github.com/tonsky/FiraCode) 字体，同样可以使用 `homebrew` 安装：

```zsh
$ brew tap homebrew/cask-fonts

$ brew install font-fira-code
```

安装完成后，依次选择 **Preferences -> Profiles -> Text -> Font**，选择 **Fira Code**：

![选择 Fira Code 字体](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_19-17-28.png)

### 显示 Status Bar

依次选择 **Preferences -> Profiles -> Session**，勾选 **Status bar enabled**。

![打开 Status Bar](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_20-09-51.png)

点击旁边的 **Configure Status Bar**，依次拖拽想要显示的功能：

![配置 Status Bar](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_20-12-27.png)

最后依次选择 **Preferences -> Appearance**，将 **Status Bar** 固定在底部，默认是在顶部的。

![固定 Status Bar 到底部](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_20-15-13.png) 

现在，来看一下 **Status Bar** 的效果：

![Status Bar 效果](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_20-18-07.png)

### 修改 iTerm2 主题

依次选择 **Preferences -> Appearance**，将 iTerm2 主题修改为自带的 **Minimal**：

![修改 iTerm2 自带主题为 Minimal](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-20_20-25-48.png)

### 修改配色方案

这里我选择的是 [Snazzy](https://github.com/sindresorhus/iterm2-snazzy)，下载安装导入成功后，依次选择 **Preferences -> Profiles -> Colors**，点击 **Color  Presets**，选择 **Snazzy**：

![选择配色方案](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-21_05-21-52.png)

## zsh

我目前使用的是 macOS Big Sur 11.1 版本，zsh 版本是 v5.8。

> 从 macOS Catalina 版本开始，macOS 默认的 Shell 已经由 bash 转变成了 zsh。

### 设置 zsh 主题

### 插件

#### zsh-autosuggestions

[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) 可以帮助我们补全命令。我们可以直接使用 `homebrew` 安装它：

```zsh
$ brew install zsh-autosuggestions
```

将下面命令添加到 `~/.zshrc` 文件的最后面：

```zsh
source /usr/local/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

最后，重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

现在，我们来看看效果：

![zsh-autosuggestions 效果](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-21_05-51-45.png)

#### autojump

[autojump](https://github.com/wting/autojump) 可以帮助我们快速跳转到期望的目录。我们可以直接使用 `homebrew` 安装它：

```zsh
$ brew install autojump
```

将下面语句添加到 `~/.zshrc` 文件中：

```zsh
[ -f /usr/local/etc/profile.d/autojump.sh ] && . /usr/local/etc/profile.d/autojump.sh
```

最后，重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

#### zsh-syntax-highlighting

[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) 可以为命令行提供语法高亮。我们可以直接使用 `homebrew` 安装它：

```zsh
$ brew install zsh-syntax-highlighting
==> Downloading https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/bottles/zs
######################################################################## 100.0%
==> Pouring zsh-syntax-highlighting-0.7.1.big_sur.bottle.1.tar.gz
==> Caveats
To activate the syntax highlighting, add the following at the end of your .zshrc:
  source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

If you receive "highlighters directory not found" error message,
you may need to add the following to your .zshenv:
  export ZSH_HIGHLIGHT_HIGHLIGHTERS_DIR=/usr/local/share/zsh-syntax-highlighting/highlighters
==> Summary
🍺  /usr/local/Cellar/zsh-syntax-highlighting/0.7.1: 27 files, 164.6KB
```

将下面语句添加到 `~/.zshrc` 文件的最后：

```zsh
source /usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

最后，重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

现在，看一下效果：

![zsh-syntax-highlighting 效果](https://gitee.com/luizyao/pictures/raw/master/img/Xnip2020-12-21_06-33-49.png)



## starship

[starship](https://github.com/starship/starship) 可以自定义各种 Shell 的终端提示符，例如：Bash、PowerShell、Zsh 等。

###  安装

```zsh
$ brew install starship
```

将如下内容加入到 `~/.zshrc` 文件的最后：

```zsh
eval "$(starship init zsh)"
```

重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

### 配置

`starship.toml` 是 starship 的配置文件，默认存放在 `~/.config/` 目录。我们可以通过指定环境变量 `STARSHIP_CONFIG` 来修改它的位置。

这里我们希望所有和 starship 相关的文件都存放在 `~/.starship/` 目录中。

打开 `~/.zshrc` 文件，添加如下内容：

```zsh
export STARSHIP_CONFIG=~/.starship/starship.toml
```

重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

创建一个空的 `starship.toml` 文件：

```zsh
$ mkdir -p ~/.starship && touch ~/.starship/starship.toml
```

> **NOTE**：
>
> startship  采用的是 [TOML](https://github.com/toml-lang/toml) 格式的配置文件。

目前我的 `starship.toml` 文件内容如下（大部分使用的是默认的配置）：

```toml
[python]
# Only use the `python3` binary to get the version.
python_binary = "python3"
```

#### Logging

starship 默认将告警和错误信息写入到 `~/.cache/starship/session_${STARSHIP_SESSION_KEY}.log` 文件，其中 `STARSHIP_SESSION_KEY` 和你目前打开的终端实例有关，可以通过如下命令查看：

```zsh
$ echo $STARSHIP_SESSION_KEY
1131127079307995
```

这里，我们同样希望将其放到 `~/.starship` 目录中，修改 `STARSHIP_CACHE` 环境变量：

打开 `~/.zshrc` 文件，添加如下内容：

```zsh
export STARSHIP_CACHE=~/.starship/cache
```

重载 `~/.zshrc` 文件：

```zsh
$ source ~/.zshrc
```

