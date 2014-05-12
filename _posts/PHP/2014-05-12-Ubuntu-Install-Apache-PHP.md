---
layout: post
title: Ubuntu 下源码安装Apache2.2 和PHP5.6
category: PHP
tags: PHP
---
###Ubuntu 14.04 安装Apache2.2
1. 解压源码包  
`tar -zxvf httpd-2.26.tag.gz`
2. 编译源码包  
这里编译的时候有很多参数可以选择,下面是一些常见的参数:  
–prefix=/usr/local/apache2 //体系无关文件的顶级安装目录PREFIX ，也就Apache的安装目录。  
–enable-module=so   //打开 so 模块，so 模块是用来提 DSO 支持的 apache 核心模块  
–enable-deflate=shared   //支持网页压缩  
–enable-expires=shared   //支持 HTTP 控制  
–enable-rewrite=shared   //支持 URL 重写  
–enable-cache //支持缓存  
–enable-file-cache //支持文件缓存  
–enable-mem-cache //支持记忆缓存  
–enable-disk-cache //支持磁盘缓存  
–enable-static-support   //支持静态连接(默认为动态连接)  
–enable-static-htpasswd   //使用静态连接编译 htpasswd – 管理用于基本认证的用户文件  
–enable-static-htdigest   //使用静态连接编译 htdigest – 管理用于摘要认证的用户文件  
–enable-static-rotatelogs   //使用静态连接编译 rotatelogs – 滚动 Apache 日志的管道日志程序  
–enable-static-logresolve   //使用静态连接编译 logresolve – 解析 Apache 日志中的IP地址为主机名  
–enable-static-htdbm   //使用静态连接编译 htdbm – 操作 DBM 密码数据库  
–enable-static-ab   //使用静态连接编译 ab – Apache HTTP 服务器性能测试工具  
–enable-static-checkgid   //使用静态连接编译 checkgid  
–disable-cgid   //禁止用一个外部 CGI 守护进程执行CGI脚本  
–disable-cgi   //禁止编译 CGI 版本的 PHP  
–disable-userdir   //禁止用户从自己的主目录中提供页面  
–with-mpm=worker // 让apache以worker方式运行 
–enable-authn-dbm=shared //对动态数据库进行操作。Rewrite时需要。  

这里我编译的时候执行命如下:  
>`./configure --prefix=/usr/local/apache2.2 --enable-so`  然后再执行 `make` 和 >`sudo make install`  

3.  然后手动启动apache 服务执行  
`sudo /usr/local/apache2.2/bin/httpd`  出现下面错误:  
httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
这是后修改httpd.conf 添加ServerName localhsot 或者直接修改原有的#ServerName www.example.com  

4. 添加开机自启动 执行  
>1. `cp /usr/local/apache2.2/bin/apachectl /etc/init.d/ `  
将apache bin目录下的apachect1 复制到/etc/init.d/目录下
>2. `sudo update-rc.d -f apachect1 defaults` 添加开机自动
>3. 重新启动,执行`netstat -tap | grep httpd` 查询这个服务是否处于监听状态

###安装PHP5.6
1. 编译参数  
`./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache2.2/bin/apxs --with-mysql=/usr/local/mysql`  在编译是我遇到下面错误:  
`configure: error: xml2-config not found. Please check your libxml2 installation.` 解决办法安装 libxml2 依赖库  
`sudo apt-get install libxml2-dev`  

2. 当出现Tank you for using PHP 时表示编译成功 然后执行   
`make` `sudo make install`   
这里在安装完后在/usr/apache2.2/moudles 下有一个叫libphp5.so动态链接库

3.执行`sudo /etc/init.d/apachect1 restart` 重新启动apache


###PHP扩展安装 以PDO为例子:
1. 进入扩展文件源码目录执行:  
`/usr/local/php/bin/phpize` 生成configure 文件  
在执行这一步时我遇到了下面问题:
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.  
解决方案如下：在/usr/src源码文件夹下执行  
`sudo  wget http://ftp.gnu.org/gnu/m4/m4-1.4.9.tar.gz`  
`sudo tar -zxvf m4-1.4.9.tar.gz`  
`sudo ./configure && make && make install`  
`wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.62.tar.gz`  
`cd autoconf-2.62/`  
`sudo ./configure `   
sudo make sudo make install`
2. 然后再执行1中你的命令,然后编译文件,命令:  
`sudo ./configure --with-php-config=/usr/local/php/bin/php-config --with-pdo_mysql=/usr/local/mysql/bin/mysql_config`
这里开始时我出现了一个错误：  
`configure: error: invalid value of canonical build`
一直没有找到解决方案,但是第二天我开机在编译自己就好了.要是有知道的求告知Email:245030109@qq.com
3. 这里在--with-pdo_mysql=/usr/local/mysql/bin/mysql_config是mysql的安装目录,
同样的要是安装mysqli扩展的话还是要带这个路径

###其他的一些Linux命令
重启/关闭/开启服务：
>sudo /etc/init.d/apachect1 restart/stop/start

开启自启服务管理器:
>sysv-rc-conf  

网络查询命令:
>netstat -tap | grep httpd






 
 