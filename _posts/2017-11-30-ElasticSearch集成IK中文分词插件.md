---
layout: post
title:  "ElasticSearch集成IK中文分词插件"
date:   2017-11-30 08:15:15 +0800
categories: Hadoop
tags: Elasticsearch
author: Jeff
mathjax: true
---

* content
{:toc}


# 环境介绍
* 1 系统及使用软件    
    Centos6.8-64位系统    
    hadoop-2.6.0    
    Java-1.7.80     
    elasticsearch-2.4.6    
    ik-1.10.6    
    tomcat-7.0.82

* 2 实验注意事项<br>
    * 总体Hadoop实验环境请参考我之前的博客(此博文将集群缩减为3台)    
    * 如未说明则一律在h1主机上使用hadoop用户操作    
    * ansible的安装可参考之前HBase安装,不一定非要使用

# IK插件安装
* 1 下载IK插件<br>
    下载插件要根据自己机器的elasticsearch版本下载,网址是https://github.com/medcl/elasticsearch-analysis-ik,这里我下载1.10.6,对应我的2.4.6版本,将下载好的文件放到第二步创建的ik目录里
    
* 2 安装<br>
    cd /home/hadoop/app/elasticsearch/plugins<br>
    mkdir ik<br>
    unzip elasticsearch-analysis-ik-1.10.6.zip<br>
    rm elasticsearch-analysis-ik-1.10.6.zip<br>
    
* 3 传到其他节点<br>
ansible other -m synchronize -a "src=/home/hadoop/app/elasticsearch/plugins/ik dest=/home/hadoop/app/elasticsearch/plugins/"

* 4 重启ElasticSearch服务并测试<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"我们是中国人"}'<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_smart&pretty=true' -d '{"text":"我们是中国人"}'<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15120037977728.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120038115155.jpg)

# 自定义IK词库
* 1 创建自定义词库<br>
    首先在ik插件的config/custom目录下创建一个文件djt.dic,在文件中添加词语即可,每一个词语一行<br>
    cd /home/hadoop/app/elasticsearch/plugins/ik/config<br>
    vim xxoo.dic<br>
    mv xxoo.dic custom/
    ![](http://ov7z79pcc.bkt.clouddn.com/15120038404826.jpg)

* 2 修改配置文件<br>
    vim IKAnalyzer.cfg.xml
    ![](http://ov7z79pcc.bkt.clouddn.com/15120038668336.jpg)

* 3 将修改好的配置文件传到其他节点,然后重启ElasticSearch服务<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15120038887528.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120039191919.jpg)
    
* 4 测试<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"真牛逼"}'<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_smart&pretty=true' -d '{"text":"真牛逼"}'<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15120039461352.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120039609506.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120039725100.jpg)

# 热更新IK词库
* 1 上面的方法每次添加热词需要重启服务,非常麻烦,固我们使用热更新的方式,需要部署tomcat服务<br>
    cd /home/hadoop/app<br>
    wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-7/v7.0.82/bin/apache-tomcat-7.0.82.tar.gz<br>
    tar zxf apache-tomcat-7.0.82.tar.gz<br>
    rm apache-tomcat-7.0.82.tar.gz<br>
    mv apache-tomcat-7.0.82 tomcat<br>
    cd tomcat<br>
    bin/startup.sh
    ![](http://ov7z79pcc.bkt.clouddn.com/15120039935021.jpg)

* 2 添加热词文件<br>
    cd webapps/ROOT<br>
    vim hot.dic
    ![](http://ov7z79pcc.bkt.clouddn.com/15120040117008.jpg)

* 3 测试热词文件能否被访问<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15120040297562.jpg)

* 4 修改ik插件的配置文件<br>
    cd /home/hadoop/app/elasticsearch/plugins/ik/config<br>
    vim IKAnalyzer.cfg.xml
    ![](http://ov7z79pcc.bkt.clouddn.com/15120040483554.jpg)

* 5 将修改的文件传到其他节点,然后重启ElasticSearch服务
    ![](http://ov7z79pcc.bkt.clouddn.com/15120040664032.jpg)

* 6 测试<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"老司机"}'<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"么么哒"}'
    ![](http://ov7z79pcc.bkt.clouddn.com/15120040878182.jpg)

* 7 实时更新热词<br>
    vim /home/hadoop/app/tomcat/webapps/ROOT/hot.dic<br>
    curl 'http://h1:9200/djt/_analyze?analyzer=ik_max_word&pretty=true' -d '{"text":"吸猫人"}'
    ![](http://ov7z79pcc.bkt.clouddn.com/15120041029055.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120041127934.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120041222979.jpg)


