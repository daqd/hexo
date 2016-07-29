---
title: 使用GitHub+Hexo免费搭建博客系统
date: 2016-07-22 15:06:48
tags:
	- github
	- hexo
categories: 环境搭建
---

本着“生命不息，折腾不止”的精神，对之前的firekylin博客系统又进行了一次大换血，这次的搭建过程不会涉及过多的服务器相关命令，基本上都是一些git相关的命令。之所以选择进行大换血，是因为之前的博客系统对markdown语法支持不是太好，添加博文的时候经常出现一些莫名其妙的语法错误，致使无法添加进去。翻看一些同行的博客，还是决定将博客托管到github，还有一点，使用github的服务器，每个月还省了五十块的服务器的费用（一分钱也是爱呀）。
<!--more-->
## 关于GitHub Page
使用github创建的博客是属于静态网站博客，也就是把写好的文章生成HTML网页，然后上传到github网站，显示的也就是HTML网页，所以加载速度会很快。

## 大致步骤

- 环境搭建
- 安装配置Hexo
- 配置主题
- 配置github
- 绑定域名
- 创建和发布文章
- 配置及美化

## 环境搭建
- 安装git （因为要给github上传文章）
- 安装Node.js（因为Hexo是基于Node.js开发的）

注：具体的安装步骤可自行百度，mac下的安装方法可参考之前搭建firelylin的文章

## 安装和配置Hexo

`node`安装完之后，打开终端，键入一下命令：

```
$ npm install -g hexo-cli
```
创建一个目录，并切换到新创建的目录

```
$ mkdir hexo
$ cd hexo
```

初始化hexo

```
$ hexo init
```
安装hexo所需的依赖包

```
$ npm install
```
至此，hexo在 本地部署完成了，启动服务来看一下

```
$ hexo server 
```
在终端我们可看到启动服务的地址，复制到浏览器瞅一下

输入http://0.0.0.0:4000

## 配置主题
在终端窗口下，定位到 Hexo 站点目录下。使用 Git checkout 代码：

```
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
终端切换到hexo根目录，用你喜欢的编辑器打开配置文件 `_config.yml`

找到里面的theme配置选项，将其参数值设置为：next

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```
再次启动服务，查看是否生效了
以上操作都正确的情况下，应该已经切换到next主题了，至此本地已经搭建完毕了。

## 配置GitHub

首先，在本地，你需要安装git工具，如果没有的话，如果你没有接触过git，推荐阅读阮一峰老师的git教程，很赞的哦。

假设你的GIT工具已经搭建完，并已经注册好gitHub账号，打开浏览器我们切换到gitHub网站，手动创建一个github仓库，这里有要求：

- 仓库名称是这样的格式：username.github.io

注：username是你的github账号

远程连接github需要配置SSH，如果，之前配置过，可跳过，没有的话，请自行百度。

查看是否配置成功，需要键入以下命令：

```
ssh -T git@github.com
```
若返回以下信息，说明配置成功

> Hi daqd! You've successfully authenticated, but GitHub does not provide shell access.

SSH配置好了，下面配置你的本地Hexo

打开Hexo目录下的_config.yml，拉倒最下面
配置为这样子,只需要把codertian改为你自己的github用户名就可以了。
这种提交方式是使用http方式提交的，我个人测试的是不需要配置SSH也可以提交

下面的配置大家要注意空格，复制我的更改即可，记住一定要是https，不能为http,不然会报错

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/daqd/daqd.github.io.git
  branch: master
```

保存，cd到Hexo的根目录

依次执行下面的命令

```
  $ hexo clean
  $ hexo g
  $ hexo d
```
第一次上传可能会让你输入git的用户名和密码

如果成功的话在浏览器输入(http://daqd.github.io) 就可以访问你的博客了。把用户名换为你自己的。

## 绑定域名
### github网站绑定域名

首先，在github网站上，点击进去你创建的那个博客仓库点击`create a new file here`

文件名称：CNAME

文件内容：www.baidu.com

注：实际上，是你的网站的域名！

然后提交

来到仓库的右边点击Download zip按钮，下载下来这个仓库，把里面的CNAME文件拖到Hexo文件的Source目录下

这一步有点像一般在服务器端添加主机头。

### 域名解析
去你注册域名的网站，可能是新网，万网，易名中国等等，进入到域名管理->域名解析
添加四条A记录，两条带www和两条不带www的记录，主机的IP填以下IP：

```
192.30.252.153
192.30.252.154
```
接下来，就是等待域名解析生效了，各个域名注册商的解析生效时间各有不同。


## 创建和发布文章
到这一步，说明你的博客已经搭建完了，通过域名解析并可直接访问，先恭喜一下！

接下来，是一些使用上的步骤：

如何添加一篇文章？后台入口在哪里？

一般网站都会有一个后台，什么admin登陆进去可以XXX，hexo和github搭配，是通过本地创建一个markdown文件，编译成静态网页文件直接传到github，是没有后台的。

那么怎样创建一篇博文？还是需要命令：

```
hexo new "这里是博文的标题"

```

此时会在`Source->_post`目录下生成一个同名字的md文件，我们用编辑器打开它，mac端的推荐mou,你就可以依照markdown语法对它为所欲为了。


写完之后，我们需要将它传到github，通过以下命令：

```
hexo d -g
```

如果出现错误`*** not found: git`

键入以下命令解决：

```
npm install hexo-deployer-git --save
```

好啦，可以进入博客看看，应该已经上传进去了。

## 配置及美化
具体的一些配置，官方的文档写的很详细，附上传送门：[官方文档](http://theme-next.iissnan.com/getting-started.html)
