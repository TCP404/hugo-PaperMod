---
title: "Hexo-文件下载功能"
tags:
  - note
  - git
  - HEXO
categories:
  - [Note]
  - [Git]
  - [HEXO]
date: 2020-08-23 16:00:35
draft: false
toc: false
images:
math: true
---

So easy~

<!--more-->

## 操作方法

效果：[点击下载](/download/大话数据结构.pdf)

1. 在source目录下，新建download目录，和_posts、About、tags、categories等目录并列
2. 将你需要分享的文件或者需要展示的图片之类，统一放到该download下
    ![1](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Hexo-download/1.png)
3. 在写文章时，通过诸如 `[点击下载](/download/xx.exe)` 这样的链接，直接写入。其他，照旧
    ![2](https://cdn.jsdelivr.net/gh/TCP404/Picgo/blog/illustration-pic/Hexo-download/2.png)
4. 在 {% btn volantis主题, https://volantis.js.org/ %}下，可以这样写 `{% btn 点击下载, /download/xx.exe, fas fa-download %}`


> 注意：
> 1. 全半角不要搞错。
> 2. 必须是**压缩文件（.exe | .zip | .rar | 7z |...）**，否则跳过去直接给你展示了不会触发下载的。