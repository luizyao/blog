---
title: "【译】性能比较：Puppeteer vs Selenium vs Playwright"
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

> 直观感觉上很容易发现 Puppeteer 和 Playwright 的执行速度比 Selenium 快，但是始终缺少一个量化的比较，并且我们对 Puppeteer 和 Playwright 之间的对决也比较感兴趣。
>
> 最近在网络上发现一篇文章  [Puppeteer vs Selenium vs Playwright, a speed comparison](https://blog.checklyhq.com/puppeteer-vs-selenium-vs-playwright-speed-comparison/)，它对 Puppeteer、Playwright 和 Selenium 的执行速度做了相对必要的测试。
>
> 这里我将其翻译成中文，并在自己的机器上做相同的实验，数据不同但结论相似。

<!--more-->

当我们决定为用户提供 [Checkly's browser checks](https://www.checklyhq.com/product/synthetic-monitoring/) 服务时，

