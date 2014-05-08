---
layout: post
title: Linux用户和组管理(1)
category: Linux
tags: Linux 
---
###用户和组配置文件
1. 组配置文件/etc/group和/etc/gshadow  
   > /etc/group 保存了组信息,例如：root:x:0:第一列为组名,第二列是组密码占位符,真正密码在/etc/gshadow里,第三列表示组号(唯一值,自动增长)(这里组号从1到499为系统组,500以后的用户组),第4列组内用户列表.(后面讲到附属组会提到) 
2. 用户配置文件/etc/passwd和/etc/shadow
> 用户信息文件/etc/passwd(共有7列),例如:
prg:x:500:500:exit,exit,exit,exit:/home/prg:/bin/bash  
第一列:用户名  
第二列:用户密码占位符(正在密码在/etc/shadow)  
第三列:用户编号  
第四列:主组编号  
第五列:注释信息(命令chnf可以设置,finger查看这个信息)  
第六列:用户的家  
第七列:用户shell命令集,一般是bash  
用户密码文件/etc/shadow
3. 组管理命令
>1. groupadd 添加组 -g 添加组的同时指定组号 
例如:groupadd -g 600 php 
>2. groupmod 修改组 -g修改组号 -n修改组名  
例如:groupmod -g 601 -n php1 php(注意这里的-n -g后面是新组号和组名,一定记住参数是用来修饰选项的)
>3. groupdel 删除组  注意:当组内有用户是不能删除该组
4. 用户管理命令
>1. useradd 添加用户  
-g 添加用户时指定用户的组(如果不指定那么自动创建一个以该用户名的组,在把这个用户添加到这个组里)
-d 添加用户是指定用户的家,不指定默认为/home/username  
例如:useradd -g php1 -d /home/xx xy   
>2. usermod 修改用户
-c 修改用户注释信息  
-g 修改用户的所属组  
-l 修改用户的登录用户名
-d 修改用户的家
-u 修改用户的uid 
例如:usermod -l mysql -d /home/php2 -c mysql xy
>3. userdel 删除用户 -r 同时删除用户的家  
例如:userdel -r mysql
5. 给添加的用户设定密码  
>新添加的用户如果不设定密码的话是不能进行登录的
使用命令passwd 给用户添加密码 passwd username
6. 禁止所有用户登录
>在/etc下创建一个文件nologin
7. 用户口令设置 passwd  
>-l 锁定用户 该用户不能登录,相当于是在改用户的密码前加两个!号  
-u 解锁用户  
-S 产看用户密码状态(大写S)  
-d 删除密码,该用户不需要密码就能登录
8. gpasswd 添加或删除组成员(操作附属组) usermod -g(修改主组)
>-a 添加  -d 删除  例如:   
groupadd group1   
groupadd group2  
useradd -g group1 user1  
useradd -g group2 user2  
gpasswd -a user1 group2 添加user1到group2中  
gpasswd -a user2 group1 添加user2到group1中  
cat /etc/group 是group1和group2组后面都有用户列表了

9. su 切换用户 没有参数默认切换到root
>su username
root 切换到其他用户不需要密码,普通用户切换到任何用户都要密码

10. newgrp 切换组,可以在主组合附属组之间切换
>当用户创建文件时,创建的文件属于当前组.要访问其他组的文件时就需要切换到其他组(当然他必须在这个组中)

11. id 显示用户id 和所属组的id groups 查看用户的所属组(包括附属组)
>id username   
groups username
12. 用户资料就是前面说的/etc/passwd文件comment列
>chfn username 设定用户信息,会在comment列中用,号分割  
finger usernam 查看刚刚设定的信息


 








