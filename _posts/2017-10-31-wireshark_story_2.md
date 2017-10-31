---
layout: post
title: "Wireshark故事小记——本地连接docker中的HBase"
date: 2017-10-31
excerpt: "Wireshark故事小记——本地连接docker中的HBase"
tags: [wireshark,dns,hbase]
--- 

{% include toc.html %}

## 1. 事件回顾

&#160; &#160; &#160; &#160;最近用到HBase，为了图方便使用docker容器来运行。结果登入容器内部操作Hbase没有问题，但是在宿主机通过hbase shell连接却查不到数据。如下图所示，执行list命令，会报错误信息。于是乎掏出Wireshark瞧一瞧有什么发现。    

```
hbase(main):001:0> list
TABLE

ERROR: 6c42865b4ee8

Here is some help for this command:
List all tables in hbase. Optional regular expression parameter could
be used to filter the output. Examples:

  hbase> list
  hbase> list 'abc.*'
  hbase> list 'ns:abc.*'
  hbase> list 'ns:.*'
```

## 2. 信息介绍   

### 2.1 使用的docker-compose.xml文件   

```
version: "2"

services:
### hbase
    hbase:
        image: nerdammer/hbase
        ports:
            - "60000:60000"
            - "60010:60010"
            - "60020:60020"
            - "60030:60030"
            - "2181:2181"
        volumes:
            - ./hbase-root/hbase:/tmp/hbase-root/hbase

```   
用到的镜像是nerdammer/hbase。

### 2.2 宿主机使用的hbase-site.xml配置   
默认配置,hbase版本是1.1.12。   

```
<?xml version="1.0"?>   
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>   
<configuration>   
</configuration>
```

## 3. 蛛丝马迹   
&#160; &#160; &#160; &#160;启动Wireshark后监听。启动宿主机hbase shell。执行list命令。可疑的数据包出现了。为了遍于观察，根据协议进行了过滤。看下图。
![Alt text](/img/in-post/wireshark_story_2/destination_packets.png)
我解释下上图4个包的含义：  
 
```   
36号包：从本机192.168.1.9向DNS服务器114.114.114.114询问知不知道主机名6c42865b4ee8对应的IPv4是多少。      
37号包：从本机192.168.1.9向DNS服务器114.114.114.114询问知不知道主机名6c42865b4ee8对应的IPv6是多少。      
38号包：DNS服务器114.114.114.114向本机回复，表示不知道查询的主机名对应IPv4是多少。   
39号包：DNS服务器114.114.114.114向本机回复，表示不知道查询的主机名对应IPv6是多少。
```

  
核对了一下容器的ID是6c42865b4ee8：   

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                                                                            NAMES
6c42865b4ee8        nerdammer/hbase     "/bin/sh -c /opt/h..."   4 days ago          Up 2 hours          0.0.0.0:2181->2181/tcp, 0.0.0.0:60000->60000/tcp, 0.0.0.0:60010->60010/tcp, 0.0.0.0:60020->60020/tcp, 0.0.0.0:60030->60030/tcp   hb_hbase_1
```
登入容器内部，才发现原来容器ID和主机名称是一样的：   

```
xxxxxx-MacBook-Pro:wireshark_story_2 xxxxxx$ docker exec -i -t hb_hbase_1 /bin/bash
[root@6c42865b4ee8 bin]#
```   

## 4.验证   
&#160; &#160; &#160; &#160;在/etc/hosts中配置：    

```
127.0.0.1  6c42865b4ee8
```   
再一次执行list命令，这下执行成功了：   

```
hbase(main):001:0> list
TABLE
marker_custom_column
test_bob
2 row(s) in 0.2290 seconds

=> ["test_table_1", "test_bob"]
```   
