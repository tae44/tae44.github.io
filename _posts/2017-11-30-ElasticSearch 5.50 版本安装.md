---
layout: post
title:  "ElasticSearch 5.50 版本安装"
date:   2017-11-30 09:22:05 +0800
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
    Java-1.8.0_151     
    elasticsearch-5.5.0    
    kibana-5.5.0

* 2 实验注意事项<br>
    * 总体Hadoop实验环境请参考我之前的博客(此博文将集群缩减为3台)    
    * 如未说明则一律在h1主机上使用hadoop用户操作    
    * ansible的安装可参考之前HBase安装,不一定非要使用    
    * Java版本请自行更换

# Elasticsearch安装
* 1 下载软件到app目录并解压<br>
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.0.tar.gz<br>
    tar zxf elasticsearch-5.5.0.tar.gz<br>
    rm elasticsearch-5.5.0.tar.gz<br>
    mv elasticsearch-5.5.0 elasticsearch

* 2 创建相关目录,需要在hadoop所有节点操作<br>
    mkdir -p /home/hadoop/data/es/data<br>
    mkdir -p /home/hadoop/data/es/datalog

* 3 修改配置文件<br>
    cd elasticsearch/config<br>
    vim elasticsearch.yml<br>
```xml
cluster.name: escluster
node.name: node-0
path.data: /home/hadoop/data/es/data
path.logs: /home/hadoop/data/es/datalog
network.host: 10.211.55.6
discovery.zen.ping_timeout: 120s
client.transport.ping_timeout: 60s
discovery.zen.ping.unicast.hosts: ["h1", "h2", "h3"]
```

* 4 将配置好的软件包分发到其他节点<br>
    ansible other -m synchronize -a "src=/home/hadoop/app/elasticsearch dest=/home/hadoop/app/"
    
* 5 其他节点配置修改,只改动如下2个属性值<br>
    * h2上:<br>
        ```xml
        node.name: node-1
        network.host: 10.211.55.7
        ```
        
    * h3上:<br>
        ```xml
        node.name: node-2
        network.host: 10.211.55.8
        ```

* 6 所有elasticsearch主机安装X-Pack插件<br>
    /home/hadoop/app/elasticsearch/bin/elasticsearch-plugin install x-pack
    ![](http://ov7z79pcc.bkt.clouddn.com/15120068612285.jpg)

* 7 在h1上安装elasticsearch-head插件<br>
    * 安装必备软件,切换为root用户操作<br>
        yum -y install npm git
        
    * 切换回hadoop用户安装插件<br>
        cd /home/hadoop/app<br>
        git clone git://github.com/mobz/elasticsearch-head.git<br>
        cd elasticsearch-head<br>
        npm install<br>
        
* 8 所有节点修改elasticsearch配置文件,添加如下3行<br>
    cd /home/hadoop/app/elasticsearch/config<br>
    vim elasticsearch.yml
    ```xml
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    http.cors.allow-headers: Authorization
    ```
    ![](http://ov7z79pcc.bkt.clouddn.com/15120077457985.jpg)

* 9 在h1上修改Gruntfile.js文件,添加1行<br>
    vim Gruntfile.js
    ![](http://ov7z79pcc.bkt.clouddn.com/15120080744717.jpg)

* 10 在h1上启动elasticsearch服务,发现如下错误,我们需要修复,修复使用root用户<br>
    /home/hadoop/app/elasticsearch/bin/elasticsearch<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15120082364563.jpg)
    * vim /etc/security/limits.conf<br>
        ![](http://ov7z79pcc.bkt.clouddn.com/15120086331548.jpg)

    * vim /etc/security/limits.d/90-nproc.conf<br>
        ![](http://ov7z79pcc.bkt.clouddn.com/15120087791323.jpg)

    * vim /etc/sysctl.conf<br>
        ![](http://ov7z79pcc.bkt.clouddn.com/15120088641946.jpg)
        ![](http://ov7z79pcc.bkt.clouddn.com/15120088913456.jpg)

    * vim /home/hadoop/app/elasticsearch/config/elasticsearch.yml<br>
        ![](http://ov7z79pcc.bkt.clouddn.com/15120090392512.jpg)

    * 切换回hadoop用户启动服务进行测试,如果测试没问题,我们关闭服务,并在其他节点修改以上配置
        ![](http://ov7z79pcc.bkt.clouddn.com/15120091948200.jpg)

* 11 在所有节点启动elasticsearch<br>
    /home/hadoop/app/elasticsearch/bin/elasticsearch
    ![](http://ov7z79pcc.bkt.clouddn.com/15120107438181.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15120107592163.jpg)

* 12 启动elasticsearch-head<br>
    cd /home/hadoop/app/elasticsearch-head<br>
    npm run start
    
# 安装Kibana
* 1 在h1上下载软件并解压<br>
    cd /home/hadoop/app<br>
    wget https://download.elastic.co/kibana/kibana/kibana-5.5.0-linux-x86_64.tar.gz<br>
    tar zxf kibana-5.5.0-linux-x86_64.tar.gz<br>
    rm kibana-5.5.0-linux-x86_64.tar.gz<br>
    mv kibana-5.5.0-linux-x86_64 kibana

* 2 修改配置文件<br>
    cd kibana/config<br>
    vim kibana.yml<br>
    ```xml
    elasticsearch.url: "http://h1:9200"
    elasticsearch.username: "elastic"
    elasticsearch.password: "changeme"
    ```
    
* 3 安装X-Pack插件<br>
    /home/hadoop/app/kibana/bin/kibana-plugin install x-pack


