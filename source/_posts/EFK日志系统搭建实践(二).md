---
title: EFK日志系统搭建实践(二)
date: 2019-03-06 14:19:57
tags:
    - EFK
    - logstash
categories: EFK     
---

上一篇文章中，我们搭建了一套最简单的EFK系统。但是收集到的数据都是作为一个整体直接写入ES，这样不利于日志的管理、查询和分析。我们希望按照日志的格式来提取相关信息（时间、日志等级、产生日志的类信息等）格式化后再存入ES。所以在原有的基础上加入Logstash（与之前ES和Filebeat版本保持一致，这里使用的版本是6.6.1）来满足我们的这个需求。  
日志系统拓扑结构如下
![拓扑图](/imgs/img-logstash-01.jpg)

### 一、搭建步骤
#### 1、Logstash安装
下载
```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-6.6.1.tar.gz
```
解压
```shell
tar -zxvf logstash-6.6.1.tar.gz
```
修改配置文件
```shell
vi /config/logstash-sample.conf
```
将配置修改为如下格式
```
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
  beats {
    port => 5044
  }
}
#grok的基本语法：key用“<>”包围，并在前面加上“?”，value则使用正则来表示，
#grok在线调试http://grokdebug.herokuapp.com/，或者使用Kibana=》Dev Tools=》Grok Debugger
filter {
    grok {
        match => {
            "message" => "(?<date>\d{4}-\d{2}-\d{2}\s(?<datetime>%{TIME}))\s(?<level>[A-Z]{4,})\s+(?<class>[A-Za-z0-9/.]{4,})\s-\s(?<msg>.*)(\n(?<moreInfo>[\s\S]*))?"
        }
        overwrite => ["message"]
    }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    user => "user"
    password => "password"
  }
}

```
上述配置解析的日志格式如下(如有新日志格式，需要重新编写过滤表达式)
```log
#格式一
2019-03-12 14:31:05,545 INFO  com.njbd.jay.controller.System.ManagerController - 公钥模:10001
#格式二
2019-01-28 10:35:39,745 ERROR com.njbd.jay.exception.GlobalExceptionHandler - serverError:更新话题失败
com.njbd.jay.exception.ServerRunTimeException: 更新话题失败
	at com.njbd.jay.service.Impl.TopicServiceImpl.updateTopic(TopicServiceImpl.java:331) ~[classes/:na]
	...
#格式三
2019-03-19 11:16:31,024 INFO  [InvoiceOrderCancelJob.java:52] com.njbd.jay.job.InvoiceOrderCancelJob - InvoiceOrderCancelJob.execute.finish
```
启动Logstash
```shell
.\bin\logstash -f .\config\logstash-sample.conf
```
#### 2、修改Filebeat输出
将Filebeat输出修改到Logstash
```yml
#----------------------------- Logstash output --------------------------------
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```
启动Filebeat

### 相关资料
[Logstash过滤文章](https://blog.51cto.com/seekerwolf/2110174)
