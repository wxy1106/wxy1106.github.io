---
title: GitHub Pages + Hexo 建站
date: 2018-07-03 11:04:05
categories: code
tags:
	- GitHub Pages
	- Hexo
---
{% note default %}
之前用的博客是找的Ruby on Rails框架源码[（GitHub地址）](https://github.com/wxy1106/xyblog)搭在了VPS上，自己维护代码和服务器都相对比较麻烦，正好VPS过期也正好最近有时间折腾，于是决定拿火了很久的GitHub Pages + Hexo来搭建一个新的个人博客
{% endnote %}
<!--more-->
## 简介
GitHub Pages是GitHub提供给开发者的一套静态站点托管服务，依托于GitHub repository我们可以把自己的静态页面发布到GitHub的服务器上，其实也就是repository主页的另一种展现形式

有了托管服务相应的我们还需要一个工具来将写好的Markdown文章渲染成静态页面，同时完成博文的管理和发布等等任务，常用的博客框架有[Jekyll](https://www.jekyll.com.cn/)和[Hexo](https://hexo.io/zh-cn/)，一般会采用后者，据说是速度更快而且主题更丰富

因此相比自己搭建和维护博客，采用这一套方案主要有这么些优势：

1. 免费服务器，不用担心成本或者过期（当然有一些容量限制，[参考](https://help.github.com/articles/what-is-github-pages/)）
2. 主题以及样式可配置，专注内容，不用花过多精力在前端页面

当然也有不可避免的缺点，比如不幸遇到维护 =_=

{% asset_img git_down.png %}

前因后果写到这，下面记录自己完整的搭建步骤，以及一些注意事项

## GitHub Page部分
### 新建GitHub Page
准确的说是新建一个GitHub repository，以username.github.io命名，注意username需要保持与你的GitHub用户名一致

{% asset_img create_repo.png %}

新建成功过后，在项目主页点击Settings可以看到GitHub Pages栏目会提示站点已经发布到了`https://username.github.io/`

{% asset_img setting.png %}

此时打开这个地址可以看到你的默认主页，理论上你已经拥有了一个博客站点，只是里面什么都没有

### 第一篇Hello World

往博客中添加或修改内容是以`Git pull/push`的方式进行的，首先需要安装Git客户端将站点源码clone到本地
```
git clone https://github.com/username/username.github.io
```
如果你想修改默认的主页
```
cd username.github.io
echo "Hello World" > index.html
```
将修改push到GitHub
```
git add .
git commit -m "Initial commit"
git push -u origin master
```
然后再打开你的主页，就可以看到修改后的主页了

到这一步GitHub Page相关的设置其实就已经完成了，下面会使用Hexo来替代上述`Git pull/push`的方式发布文章和管理站点，完成下面这一步后你才会获得一个真正的个人博客

## Hexo部分
### 安装Hexo
Hexo安装之前需要先安装两个软件

- Git（上一步已经安装过了）
- Node.js

安装好后Windows用户可以在Git Bash中输入下面指令安装Hexo
```
npm install -g hexo-cli
```
### 建站
进入站点项目目录，初始化Hexo
```
cd username.github.io
hexo init
npm install
```
### 本地发布
初始化Hexo过后即可在本地发布内容测试了，首先生成静态文件
```
hexo generate (hexo g)
```
然后启动本地服务
```
hexo server (hexo s)
```
默认情况下，本地访问地址为`http://localhost:4000/`，访问这个页面你会看到Hexo默认的一篇Hello World文章，并且你的博客出现了背景图片和布局，这些都是Hexo提供的主题功能，当然这个默认主题（Landscape）还比较粗糙，后面会写到如何更换和配置主题
### 远程发布
我们最终目的是将站点内容包括文章发布到GitHub Page上，Hexo提供了非常简单的配置方案

首先安装`hexo-deployer-git`
```
npm install hexo-deployer-git --save
```
然后修改站点配置文件`_config.yml`,设置如下参数
```
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```
注意参数的缩进以及冒号后的空格需要保持一致，完成后执行部署命令
```
hexo deploy (hexo d)
```
此时Hexo会将渲染后的页面提交到GitHub上，成功后你会看到如下提示
```
To git@github.com:username/username.github.io.git
   a1fc800..43f00a0  HEAD -> master
INFO  Deploy done: git
```
打开你的站点链接，此时你会发现之前在本地预览的站点内容已经出现在了GitHub Page上

到这一步你的个人博客就已经搭建起来了，当然搭建起来和正式发布第一篇博文之间还有很多有趣的事情可以做，留到下一篇再讲
