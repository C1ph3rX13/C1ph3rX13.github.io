---
title: "Hugo 4 Blog"
date: 2023-08-14T17:01:31+08:00
draft: false
description: "Blog Installation"
url: /posts/2023-08-14/1
tags: ["Hugo","GitHub"]
---

## 0x00 Blog Installation

Hugo + Github Pages

自建博客记录以及配置信息

### 0x01 Installation

### Hugo 框架

Github：https://github.com/gohugoio/hugo

Windows Wiki：https://gohugo.io/installation/windows/

zozo：https://github.com/varkai/hugo-theme-zozo

## 0x02 Hugo Usage

### 新建博客

~~~pow
hugo new site .
~~~

### 新建文章

~~~pow
hugo new posts/data/filename.md
~~~

### 图片保存路径

~~~powershell
../../../static/images/${filename}
~~~

### 预览修改模式

~~~pow
hugo server -D
~~~

###  发布

~~~powershell
hugo --theme=zozo --baseUrl="https://C1ph3rX13.github.io" --buildDrafts
~~~

## 0x03 GitHub Pages

### 创建 `.github/workflows`

Setting - Pages - Build and deployment - Souurce - GitHub Actions - Hugo Configs

![image-20230815145247463](https://raw.githubusercontent.com/C1ph3rX13/C1ph3rX13.github.io/main/static/images/Hugo%204%20Blog/image-20230815145247463.png)

