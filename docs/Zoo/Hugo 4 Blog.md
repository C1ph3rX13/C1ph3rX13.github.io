## Installation

### Hugo 框架

Github：https://github.com/gohugoio/hugo

Windows Wiki：https://gohugo.io/installation/windows/

zozo：https://github.com/varkai/hugo-theme-zozo

## Hugo Usage

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

### 发布

~~~powershell
hugo --theme=zozo --baseUrl="https://C1ph3rX13.github.io" --buildDrafts
~~~

## GitHub Pages

### 创建 `.github/workflows`

Setting - Pages - Build and deployment - Souurce - GitHub Actions - Hugo Configs

![image-20230815145247463](https://raw.githubusercontent.com/C1ph3rX13/C1ph3rX13.github.io/main/docs/assets/images/Hugo%204%20Blog/image-20230815145247463.png)

## 图床问题

### PicGo

使用PicGo作为图床，把图片文件都上传到固定仓库的固定文件夹，但是不能按照文章分类图片

暂时使用GitHub作为图床保存图片
