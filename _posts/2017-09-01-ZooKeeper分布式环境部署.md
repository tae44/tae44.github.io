---
layout: post
title:  "ZooKeeper分布式环境部署"
date:   2017-09-01 19:08:10 +0800
categories: Hadoop
tags: ZooKeeper
author: Jeff
mathjax: true
---

* content
{:toc}


# 单节点模式
* 1 基础配置及安装JDK，请参考 https://tae44.github.io/2017/08/26/搭建hadoop伪分布集群

* 2 su到hadoop用户下进行之后的操作，首先到官方网站下载ZooKeeper安装包，也放到app目录下
	http://mirrors.cnnic.cn/apache/zookeeper/

* 3 对zookeeper安装包解压<br>
	tar zxf zookeeper-3.4.9.tar.gz
	mv zookeeper-3.4.9 zookeeper

* 4 创建数据目录
	mkdir -p /home/hadoop/data/zookeeper/zkdata
	mkdir -p /home/hadoop/data/zookeeper/zkdatalog

* 5 在ZooKeeper安装目录的conf目录下，创建一个配置文件zoo.cfg
	cd /home/hadoop/app/zookeeper/conf/
	cp zoo_sample.cfg zoo.cfg
	vim zoo.cfg
	![](http://ov7z79pcc.bkt.clouddn.com/15042653823153.jpg)

* 6 启动/关闭ZooKeeper Server，并查看进程是否启动
	bin/zkServer.sh start
	bin/zkServer.sh stop
	![](http://ov7z79pcc.bkt.clouddn.com/15042654070134.jpg)

# 分布式模式
* 1 5台机器进行初始化设置,包括创建hadoop用户、创建文件夹、安装JDK、写hosts等等，不再重复，下面操作如未说明，则需要在5台机器上分别进行。其中z4、z5机器的角色为Observer（Observer(观察者)：Observer可以接收客户端连接，将写请求转发给learder节点，但Observer不参加投票过程，只同步learder的状态。其实它的目的是为了扩展系统，提高读取状态）。
    ![](http://ov7z79pcc.bkt.clouddn.com/15042655532092.jpg)

* 2 解压软件等操作与上面相同,不再重复

* 3 创建数据目录<br>
	mkdir -p /home/hadoop/data/zookeeper/zkdata
	mkdir -p /home/hadoop/data/zookeeper/zkdatalog

* 4 修改配置文件
	cd /home/hadoop/app/zookeeper/conf
	cp zoo_sample.cfg zoo.cfg
	vim zoo.cfg

	==z1,z2,z3配置:==
	tickTime=2000
	initLimit=10
	syncLimit=5
	dataDir=/home/hadoop/data/zookeeper/zkdata
	dataLogDir=/home/hadoop/data/zookeeper/zkdatalog
	clientPort=2181
	server.1=z1:2888:3888
	server.2=z2:2888:3888
	server.3=z3:2888:3888
	server.4=z4:2888:3888:observer
	server.5=z5:2888:3888:observer

	==z4,z5配置:==
	跟上面一样，只是在最下面加一行:peerType=observer

* 5 分别在5台机器上写入myid文件
	z1: echo 1 > /home/hadoop/data/zookeeper/zkdata/myid
	z2: echo 2 > /home/hadoop/data/zookeeper/zkdata/myid
	z3: echo 3 > /home/hadoop/data/zookeeper/zkdata/myid
	z4: echo 4 > /home/hadoop/data/zookeeper/zkdata/myid
	z5: echo 5 > /home/hadoop/data/zookeeper/zkdata/myid

* 6 在每台集群上启动ZooKeeper Server
	/home/hadoop/app/zookeeper/bin/zkServer.sh start
	zookeeper启动之后，输入“jps”命令查看进程如下
	![](http://ov7z79pcc.bkt.clouddn.com/15042660682632.jpg)

* 7 通过 status 参数查看每个节点的状态
    ![](http://ov7z79pcc.bkt.clouddn.com/15042661040085.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15042661164075.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15042661289035.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15042661413026.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15042661562385.jpg)


