---
comments: true
date: 2013-08-30T00:00:00Z
tags: ['octopress', 'apache']
title: deploy octopress to your own vps
url: /blog/2013/08/30/deploy-octopress-to-your-own-vps/
author: Xianjin YE
---

起因
====

早就想将blog迁移到自己的vps上，但

1. 之前的vps还是buyvm的屌丝机子，配置太低,而且不太稳定，一直是当ssh机子来用的，不太想跑blog
2. 已经有几个小网站跑在vps上了，访问速度实在是不理想
3. 我准备想迁移的时候，在buyvm的vps居然已经被墙了

于是由于上述种种原因，便将迁移工作搁置不动了。

7月份入职后，手里有了点闲钱，便准备入手个linode的vps，不过之前已经有好友在用linode的vps，个人觉得用linode就跑个
blog太浪费资源了。便和商量一起担负linode的vps费用。于是，机子备好，只剩代码跑起来了。

部署
====

## 客户端发布
octopress发布其实简单，其本身就自带了rsync同步部署的方式。在配置文件里写好你的vps帐号就可以完成配置了。具体可以参考[这篇文章](http://www.xiaozhou.net/deploy-octopress-to-your-vps-2013-08-13.html)。

## web server配置
上面那边参考文章在server端使用了nginx做为web server,这其实是个比较不错的选择，如果:

1. 你的vps内存比较少，配置比较低。nginx占用的资源比较少
2. 你自己独享vps或者别人没有使用其他web server的需要。 nginx的做为web server的基本功能是齐全的

但博主这个vps已经有apache的服务跑在上面了，于是就改用apache作为web server.

## 波折
机子自带的apache的版本是2.2.15,且博主想把documentRoot放在非Apache的默认路径下，结果一直报403 forbidden错误。具体解决方案可参照下面两个网址：

1. [stackoverflow 上的问答](http://stackoverflow.com/questions/10873295/error-message-forbidden-you-dont-have-permission-to-access-on-this-server)
2. [noah上的文章](http://www.noah.org/wiki/Apache2_VirtualHost_403_error)

但上面noah上提出的解决方案有缺陷，apache只需要到documentRoot目录下所有的父目录的可执行权限就可以了。也就是说如果你的Documentroot在/path/to/www这个路径，
则/path, /path/to这两个路径需要提供给Apache可执行权限。


