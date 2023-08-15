---
title: "Hugo 4 Blog"
date: 2023-08-14T17:01:31+08:00
draft: false
description: "Blog Installation"
url: /posts/2023-08-14/1
tags: ["Hugo","GitHub"]
---

# 0x00 Blog Installation

Hugo + Github Pages

# 0x01 Installation

Github：https://github.com/gohugoio/hugo

Windows Wiki：https://gohugo.io/installation/windows/

zozo：https://github.com/varkai/hugo-theme-zozo

# 0x02 Usage

新建文章

~~~pow
hugo new posts/文件名.md
~~~

图片保存路径

~~~powershell
../../static/images/${filename}

# 设置单独保存的位置
E:\C1ph3rX13 Blog\static\images\Hugo 4 Blog
~~~

预览修改模式

~~~pow
hugo server -
~~~

发布

~~~powershell
hugo --theme=zozo --baseUrl="https://C1ph3rX13.github.io" --buildDrafts
~~~
