---
layout: post
title: Vim PHP IDE
category: life
tags: life
---
下载包[vim-ide](http://yunpan.cn/QCazIUeHRW4Le) 提取码59cd

如果没有安装ctags 这安装解压包里的ctags 然后修改压缩包里.vim/plugin/taglist.vim文件改为自己的ctags安装路径. 在69行 

如果是安装了ctags且配置到path变量中那么可以直接删除taglist.vim,在69行  


这个已经删除了NERD_Tree.vim中的call s:initVariable("g:NERDTreeDirArrows", !s:running_windows) 已经去掉了！号解决目录乱码问题

一些快捷键 ：  < Leader > 为 ；

NERD_Tree： < F2 >

Taglist  : < Leader >tl  

Dox      : < Leader >fg  

DoxAuthor: < Leader >ag

DoxLic   : < Leader >lg

具体技术blog参见这里 很详细的！ [BLOG](http://www.yangyangwithgnu.net/use_vim_as_ide/):										