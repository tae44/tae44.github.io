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

3. 则会出现JRE Definition，选择Standard VM,点击Next
![](http://ov7z79pcc.bkt.clouddn.com/15038013123223.jpg)

4. 点击Directory按钮，找到你安装JDK的目录（在JAVA文件夹下），选择jdk的文件夹点确定，回到JRE Definition窗口点击Finish。

5. 然后在右面的列表里在你刚配置的JDK前打上勾，再点OK，JDK配置完毕。
![](http://ov7z79pcc.bkt.clouddn.com/15038014539847.jpg)


