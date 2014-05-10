---
layout: post
title: Ubuntu 14.04 安装 Mysql5.6服务
category: PHP
tags: PHP
---
###Ubuntu 14.04手动安装Mysql 5.6
1. 在官网下载对应linux的安装包[Mysql](http://www.mysql.com),然后执行命令
>tar -zxvf tar.gz[Mysql 压缩包名] 解压安装包   
我这里是mysql-5.6.16-linux-glibc2.5-i686.tar.gz
2. 添加一个用户和分组用于启动mysql,执行下面命令
>groupadd mysql  
useradd -g mysql mysql
3. 我们将mysql安装到/usr/local/mysql中,复制mysql-5.6.16-linux-glibc2.5-i686到/usr/local/mysql,命令:
>cp -rv mysql-5.6.16-linux-glibc2.5-i686 /usr/local/mysql
4. cd 到/usr/local/mysql,命令
>cd /usr/local/mysql
5. 开始安装mysql,命令
>scripts/install_mysql_db --user=mysql [刚才添加的用户]
6. 执行第五步时我遇到错误
>Installing MySQL system tables..../bin/mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory  
这是因为mysql需要一个依赖包执行命令  
>sudo apt-get install libaio-dev  
当在安装是出现下面信息表示安装成功了   
2014-05-09 20:34:55 2847 [Note] InnoDB: Shutdown completed; log sequence number 1625987 OK
You can start the MySQL daemon with:
cd . ; ./bin/mysqld_safe&
7. 然后我们执行bin/mysqld_safe启动mysql后执行下面命令查看mysql是否处于监听状态
>netstat -tap | grep mysql   
出现下面信息表示成功:  
tcp6    0   0 [::]:mysql              [::]:*              LISTEN      -
8. 然后执行bin/mysql就可以登录到mysql

####添加开机自启动
*我们使用手动安装的时候是没有添加开机自启动的,使用命令安装就有开机自启动.这时如果我们输入`service mysql --status`会出现提示`mysql: unrecognized service`提示*

1. Ubuntu在启动时有六种级别,默认是第二种级别.修改Ubuntu启动级别如下:
ubuntu下面没有 /etc/inittab 这个文件。用 upstart 代替原来的sysinit，进行服务进程的管理。
在 /etc/event.d/rc-default 中可以看到ubuntu默认启动的是runlevel 2，而且为了向前兼容，rc-default先检测inittab文件是否存在，如果存在，读取其中/^id:[0-9]*:initdefault:/ 行的值来启动。所以，可行的方法是：修改 rc-default 文件，将2改成其它数字。或者创建/etc/inittab添加一行`id:N:initdefault:` N是启动级别
2. Ubuntu各种级别含义  
0代表关机(halt)  
1级别是单用户模式(single)  
2级别是多用户级别，这个是默认级别  
3,4,5未定义，可以提供给用户定义其他多用户级别  
6代表重启(restart)  
S级别系统内部定义的单用户恢复模式。
3. 这里给出centos中系统启动级别  
0代表关机  
1单用户模式 2多用户模式 3字符界面 4不知道 5图形界面 6重启
CentOS 中存在/etc/inittab文件
4. 然后我们执行下面名将mysql服务添加到自启动执行命令  
>1. 先将/usr/local/mysql/support-file/mysql.server 复制到 /etc/init.d/下面  
`cp /usr/local/mysql/support-file/mysql.server /etc/init.d`
>2. 然后执行命令  
sudo update-rc.d -f mysql.server defaults  
然后会出现提示:
/etc/rc0.d/K20mysql.server -> ../init.d/mysql.server
/etc/rc1.d/K20mysql.server -> ../init.d/mysql.server
/etc/rc6.d/K20mysql.server -> ../init.d/mysql.server
/etc/rc2.d/S20mysql.server -> ../init.d/mysql.server
/etc/rc3.d/S20mysql.server -> ../init.d/mysql.server
/etc/rc4.d/S20mysql.server -> ../init.d/mysql.server
/etc/rc5.d/S20mysql.server -> ../init.d/mysql.server
然后就添加成功了,开机启动直接输入netstat -tap | grep  mysql 或者是 service mysql --status<br/>  
*这里和CentOS不一样,Ubuntu是不支持chkconfig的,在CentOs 中执行复制mysql.server到/etc/init.d/mysql下,执行chkconfig add mysql 和 chkconfig mysql on 就可以了*  
<br/>
这里在添加完后我们查看 /etc/rc2.d文件,可以看到S20mysql.server 这里S表示开机自启动(start) K表示不启动

5. 移除开机自启动
>sudo update-rc.d -f mysql.server remove  
显示信息:  
Removing any system startup links for /etc/init.d/mysql.server ...  
   /etc/rc0.d/K20mysql.server  
   /etc/rc1.d/K20mysql.server  
   /etc/rc2.d/S20mysql.server  
   /etc/rc3.d/S20mysql.server  
   /etc/rc4.d/S20mysql.server  
   /etc/rc5.d/S20mysql.server  
   /etc/rc6.d/K20mysql.server  
再次查看rc2.d文件S20mysql.server就不存在了

###使用命令安装mysql服务
参考这个[link](https://help.ubuntu.com/12.04/serverguide/mysql.html),使用命令安装后默认是开机自启动的









