---
title: ELK与EFK日志收集系统
date: 2019-03-01 10:12:38
tags:
    - 日志系统
    - ELK
    - EFK
categories: EFK     
---

## 日志收集系统

### ELK
E：Elasticsearch  
L：Logstash  
K：Kibana
<!--more-->
问题：logstash资源占用太高

### EFK
E：Elasticsearch  
F：Filebeat(Fluentd、Flume)  
K：Kibana    

ELK 和 EFK 两个区别主要是logstash、Filebeat和Fluentd的相互之间的替代(也可以一起使用)  
多种组合方式

| 对比 | 优势 | 劣势 | 备注 |
| :--------: | ----------- | -------- | -------- |
| Logstash | 灵活度高<br>插件多<br>处理问题范围广 | 与其他替代品对比性能低<br> 资源占用高(默认堆为1GB)<br>不支持缓存 | Redis或Kafka作为中心缓存池|
| Filebeat | 资源占用低<br>可靠性高<br>性能高<br>日志过滤(5.x)  | 应用范围小<br>只支持将日志发送到ES、Logstash、redis(5.x)和kafka(5.x) | 一般选择Kafka作为下游管道<br>如果选择Logstash还是会出现性能和资源消耗问题|
| Fluentd | 插件类型全面<br>可实现轻松对接<br> JSON数据格式<br>多语言接入方便 | JSON格式灵活性差<br>缓存只在输出端<br>大节点性能受限 |


### 几个问题

1. 日志结构：日志级别+IP地址+项目名称+版本号+时间戳+请求方IP地址+接口名称+参数+错误信息(可选)？？？  
日志级别：DEBUG、INFO、ERROR ？？？  
e.g.
```
     2019-03-05 12:00:00 192.168.0.2 getUserInfo:{userId: 1} errorInfo:{/* errorMsg */}
    [ERROR] 11.1.0.1 JayHomeApi v2.0 2019-03-05 13:00:00 192.168.1.1 getTopicList:{themeId: 1} errorInfo:{/* errorMsg */}
```
2. 日志存储：ES ？？？
3. 日志收集：Filebeat=》Kafka=》Logstash ？？？


### 相关资料
- [日志采集工具Logstash、Filebeat、Fluentd、Logagent、logtail、rsyslog、syslog-ng对比](http://www.amd5.cn/atang_3847.html)  
- [logstash替代方案](https://www.jianshu.com/p/8384f6cd0f22)