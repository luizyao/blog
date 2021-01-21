---
title: "【译】性能比较：puppeteer vs playwright vs selenium"
date: 2021-01-20
summary: ""
featured: false
draft: true 
toc: true
featureImage: "https://gitee.com/luizyao/pictures/raw/master/img/header-image.png"
thumbnail: ""
shareImage: ""
categories:
  - "笔记"
tags:
  - "puppeteer"
  - "playwright"
  - "selenium"
---

> 直观感觉上很容易发现 puppeteer 和 playwright 的执行速度比 selenium 快，但是始终缺少一个量化的标准，并且我们对 puppeteer 和 playwright 之间的对决也比较感兴趣。
>
> 最近在网络上发现一篇博客：[Puppeteer vs Selenium vs Playwright, a speed comparison](https://blog.checklyhq.com/puppeteer-vs-selenium-vs-playwright-speed-comparison/)，它对 puppeteer、playwright 和 selenium 的执行速度做了相对必要的测试。
>
> 这里我将其翻译成中文，并在自己的机器上做相同的实验，数据不同但结论相似。

<!--more-->

当我们决定为用户提供 [browser checks](https://www.checklyhq.com/product/synthetic-monitoring/) 服务时，我们选择 [puppeteer](https://github.com/puppeteer/puppeteer/)（一个开源的无头浏览器自动化工具）来支撑这个功能，随后又添加了 [playwright](https://github.com/microsoft/playwright) 工具。这项服务旨在通过综合的监控与测试，让用户在任何时候都能感知到他们的网站是否在正常工作。因此，测试执行的速度是我们主要的关注点。

然而，确定哪种自动化工具执行的更快并不是一件简单的事情。因此，我们决定执行我们自己的基准测试，看看 puppeteer 和 playwright 如何与传统的 [WebDriverIO](https://webdriver.io/)（使用 selenium 和 [DevTools 协议](https://webdriver.io/docs/automationProtocols.html)）进行比较。

在我们的基准测试中还有一些出人意料的发现，例如：对于较短的脚本，puppeteer 的执行速度有着显著的优势，以及 WebDriverIO 在复杂的脚本中显示出了比预期更大的变异性。阅读以下内容，以获取更详细的测试结果和过程。

## 为什么要比较这些自动化工具？

puppeteer/playwright 和 selenium 这些工具的适用范围不同，在比较执行的速度之前，任何评估它们的人都应该先了解它们之间的差异。

尽管我们中的大多数人已经使用 selenium 很多年了，我们还是迫切的想知道这些新的工具是否确实更快一些。

还需要注意的是，WebDriverIO 是一个更高层级上的框架，具有大量有用的功能，可以在多种浏览器上使用对应的工具实现自动化。

之前的经验告诉我们，使用 javascript 的 selenium 用户选择 WebDriverIO 来驱动他们的自动化脚本，因此我们也选择 WebDriverIO 而不是其它候选工具。另外，我们对 selenium 在新的 [DevTools 模式](https://webdriver.io/blog/2019/09/16/devtools.html) 上的使用也非常感兴趣。

另一个对于我们比较重要的目标是，看看我们最近在 Checkly 上开始[支持的 playwright 工具](https://blog.checklyhq.com/we-now-support-playwright/) 与一直钟爱的 puppeteer 相比如何。

## 本次基准测试的方法

如果你想直接了解测试的结果，请跳过这一节。不过我们仍然建议你在这里花些时间，以便于更好的理解测试结果的确切含义。

### 基本条件

为了使这些工具在尽可能相同的环境中进行测试，我们约定了以下基本条件：

1. **资源平等**：每个测试都是在同一台“静止的”机器上依次执行的，也就是说，在基准测试期间，后台没有发生繁重的工作负载。
2. **简单执行**：脚本的执行方式如每个工具的“入门”文档所示。例如：playwright 的 `node script.js`，并且尽量使用默认的命令行选项。

3. **相似的脚本**：我们尽量减小基准测试中不同工具使用的脚本之间的差异。尽管如此，为了脚本稳定地执行，仍然需要添加、减少或者调整一些必要的指令。
4. **最新版本**：我们测试的这些工具都使用目前为止最新发布的版本。
5. **相同的浏览器**：所有的脚本都在无头模式的 Chromium 浏览器上执行。

所有条件的细节，请参阅下一节。

### 技术参数

对于每一个基准测试，我们从同一个脚本的**1000次成功连续的执行**中收集数据。

在 selenium 的基准测试中，我们的脚本运行在一个独立的服务器上，也就是说，我们并没有像某些框架那样，每次执行都从头开始打开一个新的服务器（尽管我们每次都是用一个干净的会话）。我们这么做是为了限制执行时间上的开销。

我是在一台 Ubuntu 20.10 x64 的 vultr 云主机上运行所有的测试脚本，详细的参数如下：

 ```bash
luizyao@luizyao:~$ lscpu
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   40 bits physical, 48 bits virtual
CPU(s):                          1
On-line CPU(s) list:             0
Thread(s) per core:              1
Core(s) per socket:              1
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       GenuineIntel
CPU family:                      6
Model:                           61
Model name:                      Intel Core Processor (Broadwell, no TSX, IBRS)
Stepping:                        2
CPU MHz:                         2394.454
BogoMIPS:                        4788.90
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       32 KiB
L1i cache:                       32 KiB
L2 cache:                        4 MiB
L3 cache:                        16 MiB
 ```

