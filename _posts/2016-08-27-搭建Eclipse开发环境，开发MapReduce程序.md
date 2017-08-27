---
layout: post
title:  "搭建Eclipse开发环境，开发MapReduce程序"
date:   2017-08-27 10:30:00 +0800
categories: Hadoop
tags: Eclipse
author: Jeff
mathjax: true
---

* content
{:toc}


# JDK安装配置及MyEclipse安装这里不在赘述了


# 在MyEclipse中配置JDK
1. 打开MyEclipse然后在工具栏上找到window --> preferences

2. 在打开的窗口中找到Java --> Installed JREs 这时大家会看到窗口右侧有一个MyEclipse自带的JDK，先不用管它，按下Add...按钮
![](http://ov7z79pcc.bkt.clouddn.com/15038012222066.jpg)

3. 则会出现如图窗口，选择Standard VM,点击Next
![](http://ov7z79pcc.bkt.clouddn.com/15038013123223.jpg)

4. 点击Directory按钮，找到你安装JDK的目录（在JAVA文件夹下），选择jdk的文件夹点确定，回到JRE Definition窗口点击Finish

5. 然后在右面的列表里在你刚配置的JDK前打上勾，再点OK，JDK配置完毕
![](http://ov7z79pcc.bkt.clouddn.com/15038014539847.jpg)

# 在MyEclipse上安装hadoop插件
1. 下载2.6.0版本的hadoop插件    
http://hadoop.f.dajiangtai.com/extralib/hadoop-eclipse-plugin-2.6.0.jar

2. 把插件放到MyEclipse安装目录的dropins目录下，Windows系统下直接放入即可，Mac系统需要进入软件包内放置
![](http://ov7z79pcc.bkt.clouddn.com/15038021786654.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038022198690.jpg)

3. 重启MyEclipse，之后打开Windows --> Preferences，在窗口左侧会有Hadoop Map/Reduce选项，点击此选项，在窗口右侧设置Hadoop安装路径。
![](http://ov7z79pcc.bkt.clouddn.com/15038023306336.jpg)

4. 下载Windows下hadoop运行补丁，Mac下无需此操作，Hadoop2.6.0版本的Windows安装的是64位JDK则下载<br>
http://hadoop.f.dajiangtai.com/extralib/hadoop2.6.0-64-bin.rar

5. 解压下载的补丁，将里面hadoop.dll和winutils.exe两个文件放到Hadoop安装bin目录下，比如说是你安装在D:\hadoop-2.6.0\bin，安装包的名称不一定要跟此文一样。与此同时，还需要将hadoop.dll和winutils.exe这两个文件放入到C:\Windows\System32目录下

6. 配置Map/Reduce Locations
* 打开Windows --> show view --> Other，找到Map/Reduce选项，点击OK
![](http://ov7z79pcc.bkt.clouddn.com/15038027703061.jpg)
* 点击Map/Reduce Location选项卡，点击右下小象图标，打开Hadoop Location配置窗口
![](http://ov7z79pcc.bkt.clouddn.com/15038028928947.jpg)
* 输入Location Name，任意名称即可。配置Map/Reduce Master和DFS Mastrer，Host和Port配置成与core-site.xml的设置一致即可，然后点击Finish按钮
![](http://ov7z79pcc.bkt.clouddn.com/15038030887724.jpg)
* 如果连接成功，在project explorer的DFS Locations下会展现hdfs集群中的文件
![](http://ov7z79pcc.bkt.clouddn.com/15038031777169.jpg)

# Hadoop环境配置
![](http://ov7z79pcc.bkt.clouddn.com/15038033326343.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038033448673.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038033541701.jpg)

# 构建Map/Reduce项目





