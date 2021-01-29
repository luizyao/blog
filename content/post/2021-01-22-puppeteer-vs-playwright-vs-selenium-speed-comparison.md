---
title: "速度对比测试：puppeteer vs playwright vs selenium"
date: 2021-01-22
summary: ""
featured: false
draft: true 
toc: true
featureImage: ""
thumbnail: ""
shareImage: ""
categories:
  - "笔记"
tags:
  - "puppeteer"
  - "playwright"
  - "selenium"
  - "基准测试"
---

直观感觉上很容易发现 puppeteer 和 playwright 的执行速度比 selenium 快，但是始终缺少一个量化的对比，并且我对 puppeteer 和 playwright 之间的对决也比较感兴趣。

最近在网络上看到一篇文章：[puppeteer vs selenium vs playwright, a speed comparison](https://blog.checklyhq.com/puppeteer-vs-selenium-vs-playwright-speed-comparison/)，作者是 [Checkly 团队](https://www.checklyhq.com/), 他们对 puppeteer、playwright 和 selenium 的执行速度做了相对必要的测试。

我在自己的设备上做了类似的测试，有些结论不太一样，大家可以和原文对照阅读。

<!--more-->

不可避免的，我的测试条件和原文有所不同：

1. 执行测试的设备不同，原文是在一台 MacBook Pro 上执行的测试，我的是一台 MacBook Air；
2. 脚本测试的网站不同，原文访问的是国外的一个网站，为了避免网络延迟带来的影响，我访问的是自己内网[搭建的 pi-hole 服务](/blog/post/2020-12-06-pi-hole-getting-started-on-raspberry-pi)；
3. 依赖库使用的截止到目前最新的版本；
4. 我还额外测试了 playwright-python 工具。

> playwright-python 是 playwright 的官方 python 版本，和 playwright 一样都是由微软团队维护。
>

## 测试对象

- [puppeteer](https://github.com/puppeteer/puppeteer)
- [playwright](https://github.com/microsoft/playwright)
- WebDriverIO-selenium（[WebDriver 协议](https://webdriver.io/docs/automationProtocols.html#webdriver-protocol)）
- WebDriverIO-DevTools（[DevTools 协议](https://webdriver.io/docs/automationProtocols.html#devtools-protocol)）

> [WebDriverIO](https://webdriver.io/) 是一个 Node.js 的自动化测试框架，适用于浏览器和移动端。

## 测试方法

如果你想直接了解测试结果，请跳过这一节。不过我仍然建议你继续阅读，以便于更好的理解测试结果的确切含义。

### 基本条件

为了使测试对象尽可能的在相似的环境中运行，我们约定了以下基本条件：

1. **资源平等**：每个测试都是在同一台“静止的”机器上依次执行的，也就是说，在测试期间，后台没有发生繁重的工作负载。
2. **简单执行**：脚本的执行方式如每个工具的“入门”文档所示。例如：playwright 的 `node script.js`，并且尽量使用默认的命令行选项。
3. **相似的脚本**：尽量减小测试中不同工具之间编写脚本的差异。尽管如此，为了脚本稳定地执行，仍然需要添加、减少或者调整一些必要的指令。
4. **最新版本**：测试对象都使用目前为止最新发布的版本。
5. **相同的浏览器**：所有的脚本都在无头模式的 Chromium 浏览器上执行。

所有条件的细节，请参阅下一节。

### 测试参数

每个测试脚本都**成功的连续执行1000次**。

### 衡量指标

对每个测试都衡量以下几个指标，它们都是从这1000次的执行结果中计算出来的：

- **平均执行时间**（秒）
- **标准差**（秒）：方差的算术平方根，反应执行时间的离散程度。
- **相对标准偏差**：标准差除以平均值再乘以100%，反应执行时间相对于平均值的波动程度。
- **P95**：丢弃最慢的5%的数据，剩下的耗时最长的结果，反应非极端情况下仍然较长的执行时间。

### 尚未衡量的指标

- **可靠性**：不可靠的脚本即使执行的再快也是毫无意义的。
- **并行执行**：对自动化工具而言，并行执行是一个很重要的概念。但是，本次测试首要关注的是单个脚本的执行速度。
- **非本地执行**：和并行化一样，脚本的远程执行也是一个重要的话题，但同样不在本文的讨论范围之内。
- **资源耗用**：执行测试耗用的内存和 CPU 计算同样未予考虑。

## 测试结果

下面，你会看到本次测试的汇总结果。

### 简单的场景

在这个场景中，脚本简单的执行一个 pi-hole 的快速登录操作，预计只需要几秒钟，这可以明显的区分出测试对象启动速度之间的差异。

以下是我们的汇总测试结果：

![登录 pihole 的测试结果](https://gitee.com/luizyao/pictures/raw/master/img/image-20210128214849972.png)

原文结论 puppeteer 比 playwright 有接近 30% 的速度优势，而在我这里 **puppeteer 和 playwright 的速度相差无几，前者只是比后者有些微的优势（3%左右）**。

![login pihole: playwright vs puppeteer](https://gitee.com/luizyao/pictures/raw/master/img/pihole-login-playwright-vs-puppeteer.png)

但是，playwright 比 playwright-python 有着接近 30% 的速度优势，不过 playwright-python 有着更低的变异性。

![login pihole: playwright vs playwright-python](https://gitee.com/luizyao/pictures/raw/master/img/pihole-login-playwright-vs-playwright-python.png)

puppeteer 和 playwright 比 selenium（wdio-selenium） 均有着 40% 以上的速度优势。但是，对于 WebDriverIO 测试框架，选择那种协议（WebDriver 协议或者 DevTools 协议）对执行时间并不会产生过多的影响。

![login pihole: playwright vs wdio-selenium vs wdio-devtools](https://gitee.com/luizyao/pictures/raw/master/img/pihole-login-playwright-wdio-selenium-vs-devtools.png)

直接使用 puppeteer 比封装了一层测试框架的 wdio-DevTools 有着明显的速度优势。

![login pihole: puppeteer vs wdio-devtools](https://gitee.com/luizyao/pictures/raw/master/img/pihole-login-puppeteer-vs-wdio-devtools.png)

> wdio-DevTools 对于 chrome 浏览器使用的也是 puppeteer。

### 复杂的场景

在这个场景中，脚本登录 pihole，并添加一个白名单和黑名单，然后再删除它们。

