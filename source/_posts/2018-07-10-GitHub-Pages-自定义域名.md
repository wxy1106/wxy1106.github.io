---
title: GitHub Pages 自定义域名
date: 2018-07-10 16:38:40
tags: GitHub Pages
categories: code
---
{% note default %}
GitHub Pages的默认访问域名是`https://username.github.io`，如果用来搭建个人站点当然使用自己的域名也是不错的选择，本来GitHub Pages是支持自定义域名的，但是由于官方文档没按顺序步骤写，按文档操作会有很多遗落步骤和信息，所以自己搞了一遍把详细步骤记录下来
{% endnote %}
<!--more-->

## GitHub Pages 配置
其实GitHub的配置只需要一步，在本地Hexo目录的`source`文件夹里新建CNAME文件，注意没有后缀

关于CNAME文件务必不能按官方文档放在项目根目录，不然每次`hexo d`部署都会被覆盖掉

CNAME中写入自己的域名，支持两种格式
```
- nebula.com
- www.nebula.com
```
根据个人偏好两者只能选其一，如果使用`nebula.com`，那么配置完成后不论输入`nebula.com`还是`www.nebula.com`，最终都会跳转显示为`nebula.com`，反之亦然

保存好CNAME执行`hexo g -d`部署

打开GitHub仓库会看到CNAME已经被上传到master分支的根目录了

同时`Settings`中`GitHub Pages`标签里的域名绑定信息也会被修改为CNAME里的值，可以在这里确认一遍

{% asset_img domain_name.png %}

## 域名DNS解析
转到域名注册商的控制台，以阿里云-万网为例，添加如下解析

{% asset_img dns_setting.png %}

这里的配置实现了`www.nebula.com`和`nebula.com`两种方式都能解析到GitHub Pages

首先是`www.nebula.com`，需要添加`CNMAE`记录类型，主机记录选择`www`，对应记录值设为`wxy1106.github.io`

注意三者是严格对应的，详细文档可以参考：
[https://help.github.com/articles/setting-up-a-www-subdomain/](https://help.github.com/articles/setting-up-a-www-subdomain/)

其次是`nebula.com`，需要添加`A`记录类型，主机记录选择`@`，对应记录值设为如下4个IP
- 185.199.111.153
- 185.199.110.153
- 185.199.109.153
- 185.199.108.153

这4个IP是映射到GitHub Pages服务器的，大家都一样照抄就行，详细文档可以参考：
[https://help.github.com/articles/setting-up-an-apex-domain](https://help.github.com/articles/setting-up-an-apex-domain/#configuring-a-records-with-your-dns-provider)

配置完DNS解析视情况等待10分钟或者更久，可以在bash中ping自己的域名来检验，如果能ping通即解析生效，这时就可以访问域名来打开自己的GitHub Page了