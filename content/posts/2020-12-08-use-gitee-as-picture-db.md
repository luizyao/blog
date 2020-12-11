---
title:       "使用 Gitee 搭建个人图床"
subtitle:    ""
summary:     "本文介绍如何使用 PicGo 结合 Gitee 搭建个人图床，并集成到 Typora 工具中。"
date:        2020-12-08
author:      "Luiz Yao"
image:       ""
tags:        ["工具"]
categories:  ["其它"]
draft: false
---

我平时用 Markdown 的格式写博客，使用 Hugo 管理和构建静态站点，但也会在[博客园](https://www.cnblogs.com/luizyao/)发表文章。Hugo 支持将图片存储在 `static` 目录下，例如：`static/img/`，在博客中只需使用类似 `/img/<图片名>` 的链接，就能在生成的站点中正确的引用图片。但是，这样不利于在其它平台分享博客，最好使用完成的 URL 作为图片的引用链接。本文介绍如何用 [Gitee](https://gitee.com/) 搭建我们的个人图床。

## 安装 PicGo

> 以在 macOS 上安装为例。

[PicGo](https://github.com/Molunerfinn/PicGo) 是一个快速上传图片和获取图片 URL 链接的开源工具，并且可以结合多种图床使用。

开始安装：

```bash
$ brew cask install picgo
```

> 这里使用 `brew cask` 安装，也可以下载官方仓库 release 的 dmg 文件手动安装。下载地址：<https://github.com/Molunerfinn/PicGo/releases>

PicGo 默认不支持 Gitee 图床，我们可以安装第三方插件来获取支持：

在 PicGo 的主页面【**插件管理**】中搜索 [picgo-plugin-github-plus](https://github.com/zWingz/picgo-plugin-github-plus)，然后点击【**安装**】。

![PicGo 支持 Gitee 的插件](https://gitee.com/luizyao/pictures/raw/master/img/picgo-search-plugin-support-gitee.png)

## 配置

### 新建 Gitee 仓库

我们需要新建一个 Gitee 仓库来存放我们的图片：

- 仓库模式为【**公开**】；
- 勾选【**使用 Readme 文件初始化这个仓库**】；
- 选择【**单分支模型（只创建 master 分支）**】；

然后，在【**设置**】中创建你的**私人令牌**：

![创建 Gitee 私人令牌](https://gitee.com/luizyao/pictures/raw/master/img/create-gitee-token.png)

### 配置 githubPlus 插件

选择 PicGo 的 githubPlus 插件，将仓库的信息和上一步的个人令牌填入其中：

![配置 githubPlus 插件](https://gitee.com/luizyao/pictures/raw/master/img/picgo-gitee-getting-started.png)

## 验证

这里我上传了一个图片，对应的 Gitee 仓库中也有了最新的提交信息：

![PicGo 上传图片](https://gitee.com/luizyao/pictures/raw/master/img/picgo-upload-a-image.png)

## 集成到 Typora 中

[Typora](https://typora.io/) 是一款非常好用的 Markdown 编辑器，我们可以将 PicGo 集成到 Typora 中，这样我们在引用本地或者网上的图片时候，就会自动的上传到指定的图床中，并将链接替换成我们自己图床的 URL。

![集成到 Typora 中](https://gitee.com/luizyao/pictures/raw/master/img/20201209215236.png)
