---
title: EFK日志系统搭建实践
date: 2019-03-06 14:19:57
tags:
    - EFK
categories: EFK     
---



### 一、什么是EFK
EFK是ELK的变种，主要是将日志进行聚合，方便查询。  
EFK不是一个软件，而是一系列工具的组合(一种问题的解决方案)，来解决传统方式查询日志繁琐、低效的问题。  
<!--more-->
EFK是将ELK中的Logstash替换成Filebeat，两者对比如下

 | 对比 | 优势 | 劣势 | 备注 |
 | -------- | ----------- | -------- | -------- |
 |Logstash | 灵活度高<br>插件多<br>处理问题范围广 | 与其他替代品对比性能低<br>资源占用高(默认堆为1GB)<br>不支持缓存 | Redis或Kafka作为中心缓存池|
 |Filebeat | 资源占用低<br>可靠性高<br>性能高 | 应用范围小<br>只支持将日志发送到ES、Logstash、redis(5.x)和kafka(5.x)<br>只能简单过滤数据(5.x)  | 一般选择Kafka作为下游管道<br>如果选择Logstash还是会出现性能和资源消耗问题|
### 二、EFK具体组成
E：Elasticsearch - 数据存储、搜索、分析  
Elasticsearch是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。  
F：Filebeat - 数据搜集  
K：Kibana - 数据展示  
**最简单的EFK架构图**  
![架构图](/imgs/img-efk-01.jpg)
### 三、搭建步骤
#### 1、ES安装
因为ES需要java运行环境（Java 8 及以上），所以先要安装Java 8 下载地址，下载完成后，使用工具上传到服务器
```html
https://download.oracle.com/otn-pub/java/jdk/8u201-b09/42970487e3af4f5aa5bca3f542482c60/jdk-8u201-linux-x64.tar.gz
```
解压
```shell
tar -zxvf jdk-8u201-linux-x64.tar.gz
```
修改系统环境变量
```shell
vi /etc/profile
```
在文件末尾加上如下内容
```html
JAVA_HOME=/usr/java/jdk1.8.0_181/
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```
其中JAVA_HOME后面的路径替换为自己jdk的解压路径  
按下ESC键进入命令行模式，输入以下代码,保存并退出
```shell
:wq
```
让刚刚配置的环境变量生效
```shell
source /etc/profile
```
检验jdk是否安装成功
```shell
java -version
```
出现版本号信息则表示安装成功

下载ES，这里使用的版本为6.6.1，注意ES、Filebeat以及Kibana必须使用相同的版本
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.1.tar.gz
```
解压
```shell
tar -zxvf elasticsearch-6.6.1.tar.gz
```
进入ES根目录，修改配置文件
```shell
vi config/elasticsearch.yml
```
修改为以下参数
```json
network.host: 0.0.0.0
http.port: 9200
```
Elasticsearch不能使用root用户打开，所以需要额外的一个用户来启动它
```shell
adduser elastic
#设置密码（需要输入两次）
passwd 123456789
#设置文件夹权限
chmod -R 777 /usr/efk/es/elasticsearch-6.6.1
#切换用户
su elastic
```
启动ES
```shell
#前台运行，关闭会话窗口ES会停止
./bin/elastcsearch
#后台运行
./bin/elastcsearch -d
```
如何关闭ES 
```shell
ps -ef | grep elastic
#找到进程号并杀死,假设这里为1234
kill -9 1234
```
**可能会遇到的问题**
1. 启动时内存不足
   ```
    Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x00000000c5330000, 986513408, 0) failed; error='Cannot allocate memory' (errno=12)
    #
    # There is insufficient memory for the Java Runtime Environment to continue.
    # Native memory allocation (mmap) failed to map 986513408 bytes for committing reserved memory.
    # An error report file with more information is saved as:
    # logs/hs_err_pid13544.log
   ```
   这是因为ES默认的最小内存为1G，如果是测试环境可修改为更小的值，方法如下
   进入ES根目录，修改配置文件jvm.options
   ```shell
   vi /config/jvm.options
   ```
   修改以下两个参数到合适的大小，这里我们修改为200m
   ```
   -Xms200m
   -XMX200m
   ```
2. 最大文件描述太低
   ```
   max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
   ```
   切换到root用户，修改系统限制配置([limits.conf的作用](http://www.mamicode.com/info-detail-1941785.html))
   ```shell
   vi /etc/security/limits.conf
   ```
   将如下两个配置从65535修改为65536
   ```
   * soft nofile 65536 
   * hard nofile 65536
   ```
   如果没有这两个配置就在后面加上这两个，按下ESC键，输入以下指令保存并退出
   ```shell
   :wq
   ```
3.  最大线程数太低
    ```
    max number of threads [3882] for user [elastic] is too low, increase to at least [4096] 
    ```
    切换到root用户
    方案1、进入limits.d目录下修改配置文件
    ```
    vi /etc/security/limits.d/90-nproc.conf
    ```
    将如下配置从3882修改为5000
    ```
    * soft nproc 5000
    * hard nproc 5000
    ```
    方案2、如果没有90-nproc.conf此配置文件，那么就在问题2的配置文件后面追加如下配置
    ```
    * soft nproc 5000
    * hard nproc 5000
    root soft nproc 5000
    root hard nproc 5000
    ```
4. 最大虚拟内存区域太低
   ```
   max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
   ```
   切换到root用户，修改sysctl.conf配置文件
    ```
    vi /etc/sysctl.conf
    ```
    在文件内容末尾添加如下配置
    ```
    vm.max_map_count=262144
    ```
    按ESC键，输入以下命令保存并退出
    ```
    :wq
    ```
    执行如下命令
    ```
    sysctl -p
    ```
    以上所有配置文件的修改需要root用户权限

解决完所有问题后，启动ES，并输入以下命令验证是否启动成功
```
curl 127.0.0.1:9200  
```    
如果出现以下json数据，则表示启动成功
```json
{
  "name" : "wL6HMwx",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "FRes2_jQTNydfbLcmEwHfg",
  "version" : {
    "number" : "6.6.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "1fd8f69",
    "build_date" : "2019-02-13T17:10:04.160291Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
#### 2. 安装Kibana
下载Kibana
```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-6.6.1-linux-x86_64.tar.gz
```
进入根目录，修改配置文件
```shell
vi config/kabana.yml
```
修改如下参数
```yml
server.port: 5601
server.host: 0.0.0.0
elasticsearch.hosts: ["http://localhost:9200"]
kibana.index: ".kibana"
```
保存并退出
启动Kibana
```shell
#前台运行
./bin/kibana
#后台运行
nohup ./bin/kibana
```
如何停止Kibana
```shell
fuser -n tcp 5601
#找到进程号，杀掉进程，假设这里为5678
kill -9 5678
```


#### 3. 安装Filebeat
下载Filebeat
```shell
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.6.1-linux-x86_64.tar.gz
```
解压
```shell
tar -zxvf filebeat-6.6.1-linux-x86_64.tar.gz
```
进入根目录，修改配置文件
```shell
vi filebeat.yml
```
修改如下几个配置，注意缩进（[filebeat处理多行日志](https://my.oschina.net/openplus/blog/1589846)）

```yml
#=========================== Filebeat inputs =============================

filebeat.inputs:
- type: log
  enabled: false
  paths:
    - /var/log/*.log
    - /var/log/*.out
    #- c:\programdata\elasticsearch\logs\*
  ### Multiline options   
  #不以YYYY-MM-DD格式开头的全部追加到上一行
  multiline.pattern: ^[0-9]{4}-[0-9]{2}-[0-9]{2}
  multiline.negate: false
  multiline.match: after
#============================== Kibana =====================================
setup.kibana:  
  host: "localhost:5601"
#-------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  hosts: ["localhost:9200"]  
```
启动Filebeat
```
./filebeat -c /usr/efk/filebeat/filebeat.yml
```
如何修改Filebeat默认ES索引
[官方文档](https://www.elastic.co/guide/en/beats/filebeat/6.2/configuration-template.html)

#### 5. 配置Kibana
进入Kibana
localhost:5601
按下图顺序找到配置
![Kibana配置1](/imgs/img-kibana-01.jpg)
我们可以看到Filebeat创建的索引格式为filebeat-x.x.x-yyyy.mm.dd  
这里显示的为filebeat-6.6.1-2019.03.07  
所以为了匹配这个索引，我们需要在上方输入框输入以下索引匹配格式
```html
filebeat-6.6.1-*
```
点击Next Step进入下一步配置
![Kibana配置1](/imgs/img-kibana-02.jpg)
这里我们选择@timestamp  
点击Create index pattern，然后进入 Discover面板就能看到Filebeat收集到的日志数据了

#### 6. Kibana安全
[Ngnix实现Kibana安全认证](https://www.cnblogs.com/yjmyzz/p/filebeat-turorial-and-kibana-login-setting-with-nginx.html)
[X-pack实现Kibana安全认证-收费](https://www.cnblogs.com/cjsblog/p/9501858.html)
### 相关资料
[ES、Filebeat、Kibana官网](https://www.elastic.co/cn/products)
[Kibana查询语法(Lucene语法)](https://www.cnblogs.com/xing901022/p/4974977.html)
[Kibana用户指南](https://www.elastic.co/guide/en/kibana/6.6/index.html)