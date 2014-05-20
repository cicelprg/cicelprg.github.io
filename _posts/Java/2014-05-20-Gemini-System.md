---
layout: post
titel: 分布式文件存储系统简单说明
category: java
tags: java
---
###分布式文件存储系统架构图
![](/resource/img/java/hadoop.jpg)

###细节说明

目的：本地文件上传到若干台机器上，并取回,并保证当有一个DataNode挂掉后也能取回。

关键：需要加入一个大脑（NameNode）,对要处理的文件(上传/下载)生成,生成对应的存储方案并返回给client,同时保持与DataNode的通信,检查哪些DataNode还存活(还可以用于存放数据)。

难点 : 对一个文件进行分块过后,怎么把这些数据块均匀的分配到DataNode(可以从DataNode的使用空间，一致性hash考虑，可以把DataNode结点的信息存储到树中进行筛选)

上传文件:(自己定义一套传输协议,定义msg)
1.client向NameNode发送文件大小等信息,NameNode返回一个关于该文件存储位置的list链表
2.client遍历链表读取数据和DataNode通信，存放数据块(存放的时候每个数据块存放到多个DataNode中，这样虽然数据冗余，但是当一台服务器挂掉后也能找回数据)

下载文件:
1. client和NameNode，返回一个存储信息list
2. client按照list到各个服务器查找(这里查找的时候,对于一块元数据只需要提取一次成功就OK了)。