---
title: EFK日志系统搭建实践(三)
date: 2019-03-19 14:19:57
tags:
    - EFK
    - kafka
categories: EFK     
---

上一篇文章中，我们利用Logstash对日志进行了过滤和格式化。但是在实际生产环境中，由于数据源会非常多，Filebeat收集日志的速度以及ES处理数据的速度往往无法统一，
从而造成ES节点压力过大，这时候我们就需要引入一个中间对象，来做消息的缓冲，从而降低ES的压力。  
这篇文章中我们在上一篇的基础架构上加入Kafka。

### 一、什么是Kafka
[官方介绍](http://kafka.apache.org/intro.html)
以下是百度百科对Kafka的介绍
>Kafka是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者规模的网站中的所有动作流数据。 这种动作（网页浏览，搜索和其他用户的行动）是在现代网络上的许多社会功能的一个关键因素。 这些数据通常是由于吞吐量的要求而通过处理日志和日志聚合来解决。 对于像Hadoop一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群来提供实时的消息。  

简单的说就是分布式消息传递，存储和流处理平台。其他概念可查阅[官方说明](http://kafka.apache.org/intro.html)。
![官方图解](/imgs/img-kafka-01.jpg)

<!--more-->

### 二、Kafka如何将数据写入Elasticsearch
就目前来看，有两个使用比较广泛的方案
1. Kafka-->[Logstash](https://www.elastic.co/cn/products/logstash)-->Elasticsearch
2. Kafka-->[Kafka Connect Elasticsearch](https://www.confluent.io/connector/kafka-connect-elasticsearch/)-->Elasticsearch

方案1：配置简单、可对日志进行格式化、但是资源消耗高，后续大节点可能会出现性能问题
方案2：配置复杂、因为是直接将数据写入ES，所以不支持格式化数据、资源消耗低

考虑到上手难度和日志处理能力方面，本文使用方案1  
此时日志系统架构图如下所示
![拓扑图](/imgs/img-kafka-02.jpg)

### 三、搭建步骤
#### 1、Logstash安装
[Logstash过滤文章](https://blog.51cto.com/seekerwolf/2110174)
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
上述配置解析的日志格式如下
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
