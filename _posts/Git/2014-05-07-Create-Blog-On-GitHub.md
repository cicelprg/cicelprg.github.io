---
layout: post
title: 在GitHub上创建自己的Blog
description: ""
category: GIT
tags: GIT
---
{% include JB/setup %}

1. 首先有一个[Git](https://github.com/)账号,例如我的是cicelprg,然后创建一个respository.
命名为username.github.io
2. 然后点击刚刚建立的respository,点击右下角的setting,然后在点击settings页面中的Automatic page generator,然后再点击Continue To Layouts,最后点击也面中的publish。等10分钟左右访问username.github.io就是你的blog了.详细信息可以参见这个[link](https://help.github.com/articles/creating-pages-with-the-automatic-generator).
3. 最后一步fork别人的blog然后更改一些配置变成自己的blog.主要配置文件时_config.yml.当然你也可以直接写,github服务器时通过jekyll模板引擎是将markdown语法转换为静态html的.jekyll语法什么的点击这个[link](http://jekyllrb.com/docs/plugins/).
4. 还是fork别人的blog吧,一般在github中搜索,github.io My Blog 关键字基本出来的都是Blog模板
我也是fork别人的
5. 最后window用[github for windows](http://github-for-windows.en.softonic.com/) 比[git for windows](http://msysgit.github.io/)好用些,最后如果你愿意通过git命令行创建自己的blog 参见 [here](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html).


