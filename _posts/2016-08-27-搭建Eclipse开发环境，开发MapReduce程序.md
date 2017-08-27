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
<br>
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

# Hadoop环境配置(Mac下无需配置)
![](http://ov7z79pcc.bkt.clouddn.com/15038033326343.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038033448673.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038033541701.jpg)

# 构建Map/Reduce项目
1. 打开MyEclipse，我们选择 File --> New --> Other，点击后出现如下界面，选中Map/Reduce Project，点击下一步
![](http://ov7z79pcc.bkt.clouddn.com/15038039274290.jpg)

2. 在Project name后面输入项目名称，点击Finish完成Map/Reduce项目的创建
![](http://ov7z79pcc.bkt.clouddn.com/15038039899204.jpg)

3. 把Hadoop相关的jar包导入到项目中，点击项目右键 --> Build Path -- >Configure Build Path
![](http://ov7z79pcc.bkt.clouddn.com/15038040384150.jpg)

4. 添加4个文件夹，及common下的lib即可
![](http://ov7z79pcc.bkt.clouddn.com/15038111765997.jpg)
![Snip20170827_21](http://ov7z79pcc.bkt.clouddn.com/Snip20170827_21.png)
![Snip20170827_22](http://ov7z79pcc.bkt.clouddn.com/Snip20170827_22.png)

5. 在src目录下创建一个包名比如hadoop，然后编写一个MapReduce示例程序WordCount

```java
package hadoop;

import java.io.IOException;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    // InputPath和OutputPath路径改为自己服务器的
    FileInputFormat.addInputPath(job, new Path("hdfs://h1:9000/dajiangtai/abc.txt"));
    FileOutputFormat.setOutputPath(job, new Path("hdfs://h1:9000/dajiangtai/abc-out"));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```


10. 这里我们需要下载log4j.properties文件放到src目录下，这样程序运行时可以打印日志，便于调试程序。
http://hadoop.f.dajiangtai.com/extralib/log4j.rar

11. 将自己创建的abc.txt文件上传至HDFS文件系统的/dajiangtai目录下，使用刚才的hadoop插件即可完成各类文件的操作
![](http://ov7z79pcc.bkt.clouddn.com/15038118255798.jpg)

23. 右键选择Run as，运行程序
![](http://ov7z79pcc.bkt.clouddn.com/15038119364484.jpg)

24. 查看结果
![](http://ov7z79pcc.bkt.clouddn.com/15038120379556.jpg)
![](http://ov7z79pcc.bkt.clouddn.com/15038120575685.jpg)









