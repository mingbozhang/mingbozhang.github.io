---
layout: post
title: "Wireshark故事小记——谁动了Http的host"
date: 2017-09-08
excerpt: "Wireshark故事小记——谁动了Http的host"
tags: [wireshark,http,nginx,vue]
--- 

{% include toc.html %}

#### 一、事件回顾

&#160; &#160; &#160; &#160;某日前端同事想在家里访问局域网的一台后端服务器提供的接口。为了便于访问实际上是在nginx上面配置了策略的，根据http请求报文host内容实现局域网内部机器转发。比如：xxx200xxx8080xxx.hello.com经过解析实际上访问的是192.168.1.200端口为8080所对应的服务。   

前端本地调用接口，利用proxyTable实现跨域访问。但是发现通过IP访问是成功的，通过域名的方式访问却是失败的。下面两张图是两种调用方式，上面一张是通过IP，下面一张是通过域名：
![Alt text](/img/in-post/wireshark_story_1/ip_access.png)  

![Alt text](/img/in-post/wireshark_story_1/domain_name_access.png)  


   


#### 二、采取措施
&#160; &#160; &#160; &#160;首先我通过Postman尝试了下是不是可以通过域名来访问成功。结果没有任何问题。所以请求的报文肯定有什么地方不一样所以导致Postman是成功的，但是通过proxyTable是失败的。   
&#160; &#160; &#160; &#160;为了实战下Wireshark，考虑通过抓取两种情况的请求报文进行比对，看有没有什么猫腻。    




##### 1.分析请求报文 
首先是Postman的，配置信息如图:   
![Alt text](/img/in-post/wireshark_story_1/postman_1.png)   
通过域名访问，POST请求。请求到了目标服务。

通过Wireshark监测请求报文如下：   
![Alt text](/img/in-post/wireshark_story_1/postman_2.png)   

可以正常访问。

然后是使用proxyTable的方式：   
![Alt text](/img/in-post/wireshark_story_1/proxy_table_1.png)    
Wireshark看到的报文如下：   
![Alt text](/img/in-post/wireshark_story_1/proxy_table_2.png)  
未找到被访问的服务。


&#160; &#160; &#160; &#160;通过对以上两个报文进行比较，TCP层、IP层内容是相同的。HTTP请求报文中发现host部分和uri部分是不同的。postman这种host是xx200p8080.xxx.com。proxyTable这种是localhost:8080。怪不得无法被解析，前面也提到，服务器中nginx是配置转发是根据Host内容来解析的，因为proxyTable这种传过去的是localhost:8080，自然无法解析到正确的主机上去。   

##### 2.配置proxyTable   
经过查找发现有个参数可以解决这个问题：   
changeOrigin: true   
解释：changes the origin of the host header to the target URL     
参考地址：https://github.com/chimurai/http-proxy-middleware   

##### 3.验证  
增加了参数后，再一次抓包，这下host对了，接口调到了，如下：   
![Alt text](/img/in-post/wireshark_story_1/correct_proxy_table_1.png)


 
--PS:通过proxyTable进行转发，为什么只看到了一个请求包，就是localhost:8080那个？没有xxx200xxx8080xxx.hello.com那个？难道根本没有两个请求包，http包只会有一个，代理处理的只有TCP和IP报文。