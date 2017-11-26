---
layout: post
title: "关于本博客的写作方式"
keywords: ["模板"]
description: "关于本博客的写作及使用方式"
category: "other"
tags: ["blog"]
---
{% include JB/setup %}

最近几天，弄了一个基于jekyll的静态博客，也就是本博客。博客模板来源于github(具体地址忘了，不好意思...)。经过简单的修改，就可以将该模板配置为自己的博客。

模板修改完成后，以后的工作就是只用专注于写作了，下面简单说下写作需要注意的，也算是做个笔记，方便自己今后查阅。

### 文章格式

> 使用`markdown`来进行编写，关于markdown的具体语法，可以谷歌一下

### 文章的存放位置

> 从github上拉下该博客模板后，`_posts`文件夹就是我们需要放置写好的文章的地方

### 文章的命名方式

> 时间-文章名称.文件类型名称，如：`2016-04-15-post-modle.md`，注意，`.md`就是markdown文件的文件类型名，有的地方也会见到直接用`.markdown`

### 文章内部具体配置

```
---
layout: post 				//不用修改
title: "关于本博客的写作方式"		//文章标题
keywords: ["模板"]  			//文章关键字，可以是多个
description: "关于本博客的写作及使用方式"	//关于本文章的描述  	
category: "other"			//文章的分类
tags: ["modle"]				//文章的标签，可以是多个
---
{% include JB/setup %}

```
> 上面的一段配置是需要加在每篇文章前面的，然后就可以在配置代码后面写文章了

<hr />

### 关于本地环境的搭建

对于一般人来说，了解到上面的一些就可以很顺利的写博客了，写好后通过git更新到github就可以直接浏览，我自己是一个比较喜欢折腾的人，所以希望一切都在本地弄好了后再上传上去，所以下面就说说关于本博客本地环境的搭建过程

#### 安装nodejs

这个我就不多说了，打算自己搭本地环境的应该都是搞过nodejs，不会的同学可以自行谷歌

#### 安装ruby

点击[这里](http://rubyinstaller.org/downloads/)，获取适合自己的`ruby`版本，当然，还需要安装刚才下载地址页面的`devkit`，双击就可以安装，安装的时候注意勾选`写入环境变量`。

安装完成后，在`cmd`里输入`ruby -v`，如果显示了对应的版本号，就说明安装成功

对于刚才下载解压的`devkit`，切换到devkit所在的目录，然后手动安装
`ruby dk.rb init`	`ruby dk.rb install`

#### 安装jekyll

`gem install jekyll`，执行这个命令就可以安装jekyll，不过一般会遇到麻烦。

1. 链接不上ruby源(也就是提供ruby资源的相关地址)，所以我们这里需要切换下ruby的源，执行如下两条命令即可
`gem sources -r https://rubygems.org/`
`gem sources -a https://ruby.taobao.org/`
具体参考地址：[https://ruby.taobao.org/](https://ruby.taobao.org/)

2. 证书问题。如果错误信息里包含`certificate verify failed`之类的，一般就是证书问题，那么我们可以配置下ruby证书的环境，点击[这里](https://curl.haxx.se/ca/cacert.pem)，将证书下载到本地，然后在系统的环境变量里添加一个配置，变量名为`SSL_CERT_FILE`，变量的值为刚才下载的证书所在的路径，比如我的是`G:\Ruby22\cacert.pem`

#### 启动jekyll服务

上面的问题都解决了之后，就可以执行`jekyll server`启动本地服务，预览博客了，成功启动后，在浏览器打开[http://127.0.0.1:5000](http://127.0.0.1:5000)，就可以看到你的博客长得什么样子了。

如果启动过程中报错了，如没有安装分页组件之类的，可以执行`gem install 组件名`对应安装即可

### 下一步

基本说完了，关于博客如何接入评论系统，如何配置域名跳转，以及如何在github上部署之类的问题，网上很多文章都有介绍，如果大家安装的过程中有什么问题，欢迎留言。下一步就应该是开启我的博客之旅了，祝我旅途愉快  (｡・`ω´･)


