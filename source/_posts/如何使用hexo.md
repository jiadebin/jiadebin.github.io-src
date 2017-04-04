---
title: 如何使用hexo
date: 2017-04-03 09:45:54
tags: hexo
---

配置过程参考如下博客：
- [ibruce](http://ibruce.info/2013/11/22/hexo-your-blog/)  
- jacman主题[配置](http://wuchong.me/jacman/how-to-use-jacman/)

## 问题解决
如果npm install -g hexo 失败，尝试使用淘宝的[镜像网站](http://npm.taobao.org/)，或者去hexo[官网](http://hexo.io/)。

### Quick Start
Create a new post  

	$ hexo new "My New Post"

Run server

	$ hexo server

Clean files

	$ hexo clean

Generate static files

	$ hexo generate

Deploy to remote sites

	$ hexo deploy

上述各个命令均可以使用缩写模式，即仅写出参数的第一个字母：
	$ hexo s/c/g/d
