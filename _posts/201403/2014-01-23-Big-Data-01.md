---
layout: post
title: Big Data In Action 01 vagrant 安装
description: 大数据实战系列01 vagarnt安装。
categories:
- Hadoop 
tags:
- vagrant
---


##vagrant的介绍

## 虚拟开发环境
平常我们经常会遇到这样的问题：在开发机上面开发完毕程序，放到正式环境之后会出现各种奇怪的问题：描述符少了、nginx配置不正确、mysql编码不对、php缺少模块、glibc版本太低。

所以我们就需要虚拟开发环境，我们虚拟和正式环境一样的虚拟开发环境，而随着个人开发机硬件的升级，我们可以很容易的在本机跑虚拟机，例如vmware、VirtualBox等。因此使用虚拟化开发环境，在本机可以运行自己喜欢的OS（Windows、Ubuntu、Mac等），开发的程序运行在虚拟机中，这样迁移到生产环境可以避免环境不兼容导致的莫名错误。

虚拟开发环境特别适合团队中开发环境、测试环境、正式环境不同的场合，这样就可以使得整个团队保持一致的环境，我写这一章的初衷就是为了让大家和我的开发环境保持一致，让读者和我们整个大团队保持一致的开发环境。

## vagrant
vagrant就是为了方便的实现虚拟化环境而设计的，使用ruby开发，基于VirtualBox的接口，提供了一个可配置、轻量级的便携式虚拟开发环境。使用vagrant可以很方便的就建立起来一个虚拟环境，而且可以模拟多台虚拟机，这样我们平时还可以在开发机模拟分布式系统。

Vagrant会创建一些共享文件夹，用来给你在主机和虚拟机之间共享代码用。这样就使得我们可以在主机上写程序，然后在虚拟机中运行。如此一来团队之间就可以共享相同的开发环境，就不会再出现类似“只有你的环境才会出现的bug”这样的事情。

团队新员工加入，常常会遇到花一天甚至更多时间来从头搭建完整的开发环境，而有了vagrant，只需要直接将已经打包好的package（里面包括开发工具，代码库，配置好的服务器等）拿过来就可以工作了，这对于提升工作效率非常有帮助。

vagrant不仅可以用来作为个人的虚拟开发环境工具，而且特别适合团队使用，它使得我们虚拟化环境变得如此的简单，只要一个简单的命令就可以开启虚拟之路。


#vagrant安装配置

vagrant底层用的是Virtual Box作为虚拟机，vagrant只是一个让你可以方便设置你想要的虚拟机的便携式工具，所以第一步先安裝vagrant和Virtual Box。

## VirtualBox安装
VirtualBox是Oracle开源的虚拟化系统，它支持多个平台，所以你可以到官方网站：https://www.virtualbox.org/wiki/Downloads/ 下载适合你平台的virtualbox最新版本并安装，它的安装过程都是傻瓜化安装，一步一步执行就可以完成安装。

## vagrant安装
最新版本的vagrant已经无法通过gem命令来安装，因为依赖库太多了，所以目前无法使用gem来安装，目前网络上面很多教程还是类似这样的命令，那些都是错误的。目前唯一安装的办法就是到官方网站下载打包好的安装包：http://downloads.vagrantup.com/ 他的安装过程和Virtual Box的安装一样都是傻瓜化安装，一步一步执行就可以完成安装。

>>>尽量下载最新的程序，因为VirtualBox经常升级，升级后有些接口会变化，老的vagrant可能无法使用。

检测是否安装成功，打开终端命令行工具，输入vagrant，是不是已经可以运行，如果不行，请检查PATH里面是否有vagrant所在的路径。

## vagrant配置

当我们安装好Virtual Box和vagrant后，我们要开始考虑在VM上使用什么操作系统了，一个打包好的操作系统在vagrant称为Box，即Box是一个打包好的操作系统环境，目前网络上什么都有，所以你不用自己去制作操作系统或者制作Box：[vagrantbox.es](http://www.vagrantbox.es/)上面有大家熟知的大多数操作系统，你只需要下载就可以了，下载主要是为了安装的时候快速，当然vagrant也支持在线安装。

### 建立开发环境目录
我的开发机是windows，所以我建立了如下的开发环境目录，读者可以根据自己的系统不同建立一个目录就可以：

	F:\\Boxes

### 下载box
前面讲了box是一个操作系统环境，实际上它是一个zip包，包含了vagrant的配置信息和VirtualBox的虚拟机镜像文件.我们这一次的实战使用官方提供了一个box:Ubuntu lucid 64 http://files.vagrantup.com/lucid64.box 

当然你也可以选一个自己团队在用的系统，例如centos、Debian等，我们可以通过上面说的地址下载开源爱好者们制作好的box。当然你自己做一个也行，下一节我会讲述如何自己制作包。

### 添加box
添加box的命令如下：

	vagrant box add base 远端的box地址或者本地的box文件名
	
`vagrant box add` 是添加box的命令

`base`是box的名称，可以是任意的标题，base是默认名称，主要用来标识一下你添加的box，后面的命令都是基于这个标识来操作的。

例子：

	vagrant box add base http://files.vagrantup.com/lucid64.box
	vagrant box add base https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box
	vagrant box add base CentOS-6.3-x86_64-minimal.box
	vagrant box add "CentOS 6.3 x86_64 minimal" CentOS-6.3-x86_64-minimal.box

我在开发机上面是这样操作的，首先进入我们的开发环境目录`F:\Boxes`，执行如下的命令
	
	vagrant box add base lucid64.box
	
安装过程的信息

	Downloading or copying the box...
	Extracting box...te: 47.5M/s, Estimated time remaining: --:--:--)
	Successfully added box 'base' with provider 'virtualbox'!
	
box中的镜像文件被放到了：`C:\Users\当前用户名\.vagrant.d\boxes\`目录下。

通过`vagrant box add`这样的方式安装远程的box，可能很慢，所以建议大家先下载box到本地再执行这样的操作。

### 初始化
初始化的命令如下：

	vagrant init

如果你添加的box名称不是base，那么需要在初始化的时候指定名称，例如

	vagrant init "CentOS 6.3 x86_64 minimal"

初始化过程的信息：

	A `Vagrantfile` has been placed in this directory. 
	You are now ready to `vagrant up` your first virtual environment! 
	Please read the comments in the Vagrantfile as well as documentation on `vagrantup.com` for more information on using Vagrant.

这样就会当前目录生成一个	`Vagrantfile`的文件，里面有很多配置信息，后面我们会详细讲解每一项的含义，但是默认的就可以开箱即用。

### 启动虚拟机
启动的命令如下：

	vagrant up

启动过程的信息:	

	Bringing machine 'default' up with 'virtualbox' provider...
	[default] Importing base box 'base'...
	[default] Matching MAC address for NAT networking...
	[default] Setting the name of the VM...
	[default] Clearing any previously set forwarded ports...
	[default] Creating shared folders metadata...
	[default] Clearing any previously set network interfaces...
	[default] Preparing network interfaces based on configuration...
	[default] Forwarding ports...
	[default] -- 22 => 2222 (adapter 1)
	[default] Booting VM...
	[default] Waiting for VM to boot. This can take a few minutes.
	[default] VM booted and ready for use!
	[default] Mounting shared folders...
	[default] -- /vagrant

### 链接到虚拟机
上面已经启动了虚拟机了，我们现在就可以通过ssh链接到虚拟机，我开发机可以这样链接：

![登录显示][1]

	
我们就可以像链接到一台服务器一样进行操作了。

>>>window机器不支持这样的命令，必须使用第三方客户端来进行链接，例如putty、Xshell4等.

>>>xshell为例：

>>>主机地址: 127.0.0.1

>>>端口: 2222

>>>用户名: vagrant

>>>密码: vagrant

### 系统信息
进入系统之后我们可以看一下系统的基础信息：

![系统信息][2]
	

### 配置详解
在我们的开发目录下有一个文件`Vagrantfile`，里面包含有大量的配置信息，主要包括三个方面的配置，虚拟机的配置、SSH配置、vagrant的一些基础配置。vagrant是使用ruby开发的，所以它的配置语法也是ruby的，但是我们没有学过ruby的人还是可以跟着他的注释知道怎么配置一些基本项的配置。

1. box设置

		config.vm.box = "base"
	
	上面这配置展示了vagrant要去启用那个box作为系统，也就是上面我们输入`vagrant init Box名称`时所指定的box，如果沒有输入Box名称的話，那么默认就是base，Virtual Box提供了VBoxManage这个command line工具，可以让我们设定VM，用`modifyvm`这个命令让我们可以设定VM的名称和内存大小等等，这里说的名称指的是在Virtual Box中显示的名称，我们也可以在Vagrantfile中进行设定，在Vagrantfile中加入如下这行就可以设定了：

		config.vm.provider "virtualbox" do |v|
		  v.customize ["modifyvm", :id, "--name", "fle4y", "--memory", "512"]
		end	

	这行设置的意思是调用VBoxManage的modifyvm的命令，设置VM的名称为fle4y，VM的内存为512MB，你可以类似设置来个性化设置你的VM。
	
2. 网络设置

	Vagrant有两种方式来进行网络链接，一种是host-only(主机模式)，意思是主机和虚拟机之间的网络互访，而不是虚拟机访问internet的技术，也就是只有你一個人自High，其他人访问不到你的虚拟机。另一种是Bridge(桥接模式)，该模式下的VM就像是局域网中的一台独立的主机，也就是说需要VM到你的路由器要IP，这样的话局域网里面其他机器就可以访问它了，一般我们设置虚拟机都是自high为主，所以我们的设置一般如下：

		config.vm.network :private_network, ip: "11.11.11.11"

	这里我们虚拟机设置为hostonly，并且指定了一个IP，IP的建议最好不要用`192.168..`，因为很有可能和你局域网里面的其他机器IP冲突，因为你可以尽量使用类似`11.11..`这样子的。

3. hostname设置
	hostname的设置非常简单，Vagrantfile中加入下面这行就可以了：

		config.vm.hostname = "go-app"

	设置hostname非常重要，因为当我们有很多台虚拟服务器的时候，都是依靠hostname來做识别的，例如Puppet或是Chef，都是通过hostname來做识别的，既然设置那么简单，所以我们就別偷懒，设置一个。
	
4. 同步目录
	我们上面介绍过/vagrant目录默认就是当前的开发目录，这是在虚拟机开启的时候默认挂载同步的。我们还可以通过配置来设置额外的同步目录：
	
		 config.vm.synced_folder "F:/share", "/home/vagrant/share"	

	上面这个设定，第一个参数是主机的目录，第二个是虚拟机挂载的目录
	
5. 端口转发

		config.vm.network :forwarded_port, guest: 80, host: 8080	

	这一行的意思是把Host机器上8080端口传来的数据forward到虚拟机的80端口的服务上，例如你在你的虚拟机上，使用Nginx跑了一个Go应用，那么你在host机器上在浏览器中打开`http://localhost:8080`时，vagrant就会把这个请求传到VM里面跑在80端口的Nginx服务上，因此我们可以通过这个设置来帮助我们去设定host和VM之间，或是VM和VM之间的信息交互。

>>>修改完Vagrantfile的配置后，记得要用vagrant reload的命令重启VM才能使得VM可以用新的配置




###常用链接

[![给我捐款](http://c000005.qiniudn.com/donate_me.png "给我捐款")](http://me.alipay.com/0xc000005)

[回到主页][3]

  [1]: http://c000005.qiniudn.com/1.png

  [2]: http://c000005.qiniudn.com/2.png

  [3]: http://0xc000005.github.io/