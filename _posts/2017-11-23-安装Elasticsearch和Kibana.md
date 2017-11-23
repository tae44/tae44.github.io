---
layout: post
title:  "安装Elasticsearch和Kibana"
date:   2017-11-23 16:10:13 +0800
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
    zookeeper-3.4.9    
    elasticsearch-2.4.6    
    kibana-4.6.6

* 2 实验注意事项<br>
    * 总体Hadoop实验环境请参考我之前的博客(此博文将集群缩减为3台)    
    * 如未说明则一律在h1主机上使用hadoop用户操作

# 安装Elasticsearch
* 1 下载软件并解压<br>
    cd app/<br>
    wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.4.6/elasticsearch-2.4.6.tar.gz<br>
    tar zxf elasticsearch-2.4.6.tar.gz<br>
    rm elasticsearch-2.4.6.tar.gz<br>
    mv elasticsearch-2.4.6 elasticsearch
    
* 2 创建相关目录,需要在hadoop所有节点操作<br>
    mkdir -p /home/hadoop/data/es/data<br>
    mkdir -p /home/hadoop/data/es/datalog<br>
    
* 3 修改配置文件<br>
    cd elasticsearch/config<br>
    vim elasticsearch.yml
    ```xml
cluster.name: escluster                     # 集群起名字
node.name: node-0                           # 节点起名字
path.data: /home/hadoop/data/es/data
path.logs: /home/hadoop/data/es/datalog
network.host: 10.211.55.6                   # 本机ip
discovery.zen.ping.multicast.enabled: false # 关掉多播
discovery.zen.ping_timeout: 120s
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["h1", "h2", "h3"] # 指定单播主机
```

* 4 将配置好的软件包分发到其他节点<br>
    ansible other -m synchronize -a "src=/home/hadoop/app/elasticsearch dest=/home/hadoop/app/"
    
* 5 其他节点配置修改,只改动如下2个属性值
    * h2上:
        ```xml
        node.name: node-1
        network.host: 10.211.55.7
        ```
        
    * h3上:
        ```xml
        node.name: node-2
        network.host: 10.211.55.8
        ```
        
* 6 安装marvel插件(所有节点都需要安装)<br>
    cd /home/hadoop/app/elasticsearch<br>
    bin/plugin install license<br>
    bin/plugin install marvel-agent
    ![](http://ov7z79pcc.bkt.clouddn.com/15114267174952.jpg)

* 7 安装head插件(在h1节点安装即可)<br>
    bin/plugin install mobz/elasticsearch-head
    ![](http://ov7z79pcc.bkt.clouddn.com/15114267922679.jpg)

# 安装Kibana
* 1 下载软件并解压<br>
    cd /home/hadoop/app<br>
    wget https://download.elastic.co/kibana/kibana/kibana-4.6.6-linux-x86_64.tar.gz<br>
    tar zxf kibana-4.6.6-linux-x86_64.tar.gz<br>
    rm kibana-4.6.6-linux-x86_64.tar.gz<br>
    mv kibana-4.6.6-linux-x86_64 kibana

* 2 修改配置文件<br>
    cd kibana/config<br>
    vim kibana.yml
    ![](http://ov7z79pcc.bkt.clouddn.com/15114270595695.jpg)

* 3 安装marvel插件<br>
    cd ..<br>
    bin/kibana plugin --install elasticsearch/marvel/latest
    ![](http://ov7z79pcc.bkt.clouddn.com/15114272820884.jpg)

* 4 每台机器启动elasticsearch<br>
    /home/hadoop/app/elasticsearch/bin/elasticsearch -d -p /home/hadoop/data/es/pid
    
* 5 在h1上启动kibana<br>
    nohup /home/hadoop/app/kibana/bin/kibana &<br>
    exit
    ![](http://ov7z79pcc.bkt.clouddn.com/15114273869961.jpg)

* 6 打开ElasticSearch的测试网页<br>
    http://h1:9200/<br>
    http://h1:9200/_cluster/health?pretty<br>
    http://h1:9200/_plugin/head/<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15114274696728.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15114274916158.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15114275170868.jpg)

* 7 打开Kibana的测试网页<br>
    http://h1:5601/app/marvel
    ![](http://ov7z79pcc.bkt.clouddn.com/15114276145789.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15114276465694.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15114276680075.jpg)



