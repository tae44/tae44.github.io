---
layout: post
title:  "Spark standalone 及 on YARN 的配置与部署"
date:   2018-01-17 08:30:36 +0800
categories: Hadoop
tags: Spark
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
    spark-2.1.2    

* 2 实验注意事项<br>
    * 总体Hadoop实验环境请参考我之前的博客    
    * 如未说明则一律在h1主机上使用hadoop用户操作    
    * ansible的安装可参考之前HBase博客进行安装,不一定非要使用

# Spark standalone 运行模式
* 1 使用上次编译好的软件包<br>
    cd /home/hadoop/app<br>
    tar zxf spark-2.1.2-bin-my-spark.tgz<br>
    rm spark-2.1.2-bin-my-spark.tgz<br>
    mv spark-2.1.2-bin-my-spark spark
    
* 2 指定在哪些节点上运行worker<br>
    cd spark/conf<br>
    mv slaves.template slaves<br>
    vim slaves<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15161492087261.jpg)

* 3 spark提交job时的默认配置,暂时保持默认<br>
    mv spark-defaults.conf.template spark-defaults.conf<br>
    vim spark-defaults.conf
    
* 4 spark的环境变量<br>
    mv spark-env.sh.template spark-env.sh<br>
    vim spark-env.sh
    ```xml
    export JAVA_HOME=/home/hadoop/app/jdk 
    SPARK_MASTER_WEBUI_PORT=8888
    SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=h1:2181,h2:2181,h3:2181 -Dspark.deploy.zookeeper.dir=/home/hadoop/data/zookeeper"
```

* 5 将软件包传到其他节点,并在所有节点启动zookeeper<br>
    cd ..<br>
    ansible other -m synchronize -a "src=/home/hadoop/app/spark dest=/home/hadoop/app/"<br>
    ansible all -a "~/app/zookeeper/bin/zkServer.sh start"

* 6 在第一个master的节点h1上启动集群<br>
sbin/start-all.sh

* 7 在第二个节点h2上单独启动master进程<br>
cd app/spark<br>
sbin/start-master.sh

* 8 查看启动情况<br>
cat /home/hadoop/app/spark/logs/spark-hadoop-org.apache.spark.deploy.master.Master-1-h1.out
![](http://ov7z79pcc.bkt.clouddn.com/15161495735159.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15161495903901.jpg)

* 9 访问web ui<br>
    http://h1:8888<br>
    http://h2:8888<br>
    ![](http://ov7z79pcc.bkt.clouddn.com/15161496510892.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15161496625878.jpg)

* 10 Job提交与运行<br>
    ./bin/spark-submit --master spark://h1:7077,h2:7077 --class org.apache.spark.examples.SparkPi examples/jars/spark-examples_2.11-2.1.2.jar
    ![](http://ov7z79pcc.bkt.clouddn.com/15161497243322.jpg)
    ![](http://ov7z79pcc.bkt.clouddn.com/15161497447293.jpg)

* 11 将WordCount测试项目打包并上传到服务器
    ![](http://ov7z79pcc.bkt.clouddn.com/15161498277076.jpg)

* 12 启动hdfs集群<br>
    /home/hadoop/app/hadoop/sbin/start-dfs.sh

* 13 将测试数据放到hdfs上
    ![](http://ov7z79pcc.bkt.clouddn.com/15161498545336.jpg)

* 14 Spark集群提交作业<br>
    ./spark-submit --master spark://h1:7077,h2:7077 --class com.jeff.MyWordCount /home/hadoop/app/spark/mys-1.0-SNAPSHOT.jar hdfs://cluster1/data/spark/input/spark.txt hdfs://cluster1/data/spark/output
    ![](http://ov7z79pcc.bkt.clouddn.com/15161498839000.jpg)

# Spark on YARN 配置与部署
* 1 Spark On YARN安装非常简单，只需要下载编译好的Spark安装包，在一台带有Hadoop Yarn客户端的机器上解压即可,配置HADOOP_CONF_DIR或者YARN_CONF_DIR环境变量，让Spark知道Yarn的配置信息<br>
    cd /home/hadoop/app/spark/conf<br>
    vim spark-env.sh
    ```xml
    HADOOP_CONF_DIR=/home/hadoop/app/hadoop/etc/hadoop
    ```
    ![](http://ov7z79pcc.bkt.clouddn.com/15161503718244.jpg)
    
* 2 在hadoop集群的yarn配置文件中取消yarn的内存检查(实验机器内存大的话可以不调)<br>
    cd /home/hadoop/app/hadoop/etc/hadoop<br>
    vim yarn-site.xml
    ```xml
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
        # 是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true
    </property>
  
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
        # 是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true
    </property>
```

* 3 spark-shell运行在YARN上<br>
    cd ../bin<br>
    ./spark-shell --master yarn
    ![](http://ov7z79pcc.bkt.clouddn.com/15161502971262.jpg)

* 4 提交Spark job给YARN
    ./spark-submit --master yarn --deploy-mode cluster --class com.dajiangtai.spark.MyWordCount /home/hadoop/app/spark/learning-saprk-1.0-SNAPSHOT.jar hdfs://cluster1/data/spark/input/spark.txt hdfs://cluster1/data/spark/output


