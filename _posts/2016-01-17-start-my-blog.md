---
layout:     post
title:      "Welcome to Sethbrin Blog"
subtitle:   " \"我的第一博客\""
date:       2016-01-16 00:00:00
author:     "sethbrin"
header-img: "img/post-bg-2016.jpg"
tags:
    - 生活
---

> “Yeah It's on. ”


## 前言

Sethbrin 的博客现在终于开通了。

[跳过废话，直接看搭建过程 ](#build)



2016 年，sethbrin 的博客终于开通了,很久以来都想写点东西,但是总是被一些事情干扰(PS: 其实就是懒)。

在工作和生活中,时常遇到一些事情和一些问题,当时解决了,但过些时候又忘记了,所以需要一个地方来记录生活的点点滴滴,仔细回味.

<p id = "build"></p>
---

## 正文

接下来说说搭建这个博客的搭建过程.

其实也不叫什么搭建过程,就是clone Hux大神的项目,然后改吧改吧,然后就当自己博客.下面我也简单废话几句.

1. 在github上建一个空项目,名字为`youname.github.io`

2. 从github上拷贝Hux大神的项目,到本地`git clone https://github.com/Huxpro/huxpro.github.io.git youname.github.io`

3. 修改项目的_config.yml文件,其中可能会遇到一个这样的问题:

> Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `gems: [jekyll-paginate]` in your configuration file.

只需用gem安装jekyll-paginate即可`gem install jekyll-paginate`,然后再修改_config.yml文件,添加`gems: ['jekyll-paginate']`

