---
layout: post
title: "JVM学习——编译OpenJDK"
date: 2016-01-06
excerpt: "A ton of text to test readability."
---

{% include toc.html %}  

 &#160; &#160; &#160; &#160;最近在学习《深入理解java虚拟机 第二版》这本书。书中第一部分建议大家自己编译OpenJDK。抱着学习态度也来编译个玩一玩。下面进入正题。

## 1.编译环境介绍

|名称  |  版本  |   
| ------------- |:-------------:|    
|操作系统 | CentOS Linux release 7.1.1503 (Core) |   
|Linux内核版本 |3.10.0-229.el7.x86_64 |   
|gcc版本 | 4.8.2 20140120 (Red Hat 4.8.2-16) (GCC)|   
|openJdk版本 |openjdk-7u40|   


## 2.准备工作
### 2.1下载OpenJDK
打开网站https://jdk7.java.net/source.html  
在这里复制链接直接在linux中wget。  
wget http://www.java.net/download/openjdk/jdk7u40/promoted/b43/openjdk-7u40-fcs-src-b43-26_aug_2013.zip

### 2.2 下载依赖
具体的依赖要求说明都在README-builds.html 中了，这里直接来命令：  
# yum install -y  glibc*  
# yum install cups-devel  
# yum install -y  alsa*  
# yum install -y fontconfig*  
# yum install zip*  
# yum install  libstdc++-static

解压压缩包,将得到openjdk文件夹  
unzip  openjdk-7u40-fcs-src-b43-26_aug_2013.zip  
cd openjdk  
这里有一份构建说明文档README-builds.html ，建议在windows下面看。后面步骤全靠它了。

**安装freetype**  
下载源码
# wget http://jaist.dl.sourceforge.net/project/freetype/freetype2/2.3.12/freetype-2.3.12.tar.gz  
解压源码,编译安装
# tar -zxvf freetype-2.3.12.tar.gz  
# cd freetype-2.3.12  
# ./configure && make && sudo -u root make install  

**下载并安装ANT**  
配置ANT环境变量  
# wget http://mirrors.hust.edu.cn/apache//ant/binaries/apache-ant-1.9.6-bin.tar.gz  
# vi /etc/profile  
 加入如下内容：  
export ANT_HOME=/opt/openJDKroom/apache-ant-1.9.6  
export PATH=$PATH:$ANT_HOME/bin  
# source /etc/profile  

**下载并安装CUPS**  
wget http://www.cups.org/software/2.1.2/cups-2.1.2-source.tar.bz2  
tar  -jxvf  cups-2.1.2-source.tar.bz2  
# ./configure && make && make install  

### 2.3 屏蔽已经配置的JAVA_HOME
下面贴出README-builds.html中原文：  
    Note that some Linux systems have a habit of pre-populating your environment variables for you,   for example JAVA_HOME might get pre-defined for you to refer to the JDK installed on your Linux   system. You will need to unset JAVA_HOME. It's a good idea to run env and verify the   environment variables you are getting from the default system settings make sense for building   the OpenJDK.   
 
先检查一下本地是否配置有JAVA_HOME  
# env | grep JAVA_HOME  
如果有则屏蔽掉。  


### 2.4 配置引导JDK
下面贴出README-builds.html中原文：
        
     Bootstrap JDK  
     All OpenJDK builds require access to the previously released JDK 6, this is often called a   bootstrap JDK. The JDK 6 binaries can be downloaded from Sun's JDK 6 download site. For build   performance reasons is very important that this bootstrap JDK be made available on the local   disk of the machine doing the build. You should always set ALT_BOOTDIR to point to the location   of the bootstrap JDK installation, this is the directory pathname that contains a bin, lib, and   include It's also a good idea to also place its bin directory in the PATH environment variable,   although it's not required.  

**下载安装Bootstrap**   
这里下载JDK 6 Linux 64位jdk-6u45-linux-x64.bin  
# chmod +x jdk-6u45-linux-x64.bin  
# ./jdk-6u45-linux-x64.bin  

安装完毕后，当前目录下会生成jdk1.6.0_45目录，配置ALT_BOOTDIR环境变量为该目录路径。在/etc/profile中  增加如下配置（以实际为准）：  
export ALT_BOOTDIR=/opt/openJDKroom/jdk1.6.0_45  
# source /etc/profile  

### 2.5 配置环境变量ALT_CACERTS_FILE
     Certificate Authority File (cacert)  
     See http://en.wikipedia.org/wiki/Certificate_Authority for a better understanding of the   Certificate Authority (CA). A certificates file named "cacerts" represents a system-wide   keystore with CA certificates. In JDK and JRE binary bundles, the "cacerts" file contains root   CA certificates from several public CAs (e.g., VeriSign, Thawte, and Baltimore). The source   contain a cacerts file without CA root certificates. Formal JDK builders will need to secure     permission from each public CA and include the certificates into their own custom cacerts file.   Failure to provide a populated cacerts file will result in verification errors of a certificate   chain during runtime. The variable ALT_CACERTS_FILE can be used to override the default   location of the cacerts file that will get placed in your build. By default an empty cacerts   file is provided and that should be fine for most JDK developers.  

进入openjdk目录中,查找名称为cacerts的文件：  
# cd openjdk  
# find ./ -name cacerts  
显示结果如下：  
[root@localhost openjdk]# find ./ -name cacerts  
./jdk/src/share/lib/security/cacerts  
打开/etc/profile，将以下内容加入（具体路径根据实际情况）：  
export ALT_CACERTS_FILE=/opt/openJDKroom/openjdk/jdk/src/share/lib/security/cacerts  

## 3.开始编译
终于要开始编译了！\>_<    
README-builds.html中用的是gmake，实际和make是一样的，下面是解释：  

    gmake是GNU Make的缩写。  
    Linux系统环境下的make就是GNU Make，之所以有gmake，是因为在别的平台上，make一般被占用，GNU make只  好叫gmake了。
      
进入到openjdk根目录下，进行编译   
cd /opt/openJDKroom/openjdk   
gmake sanity   
gmake   
结果执行报错：   
{% highlight java %}
gmake[6]: Entering directory `/opt/openJDKroom/openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2/product'  
gmake[6]: stat: libjvm.so: Too many levels of symbolic links
 {% endhighlight %}

进入到/opt/openJDKroom/openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2/product目录下有下面链接：  
libjvm.so->libjvm.so  
libjvm.so.1->libjvm.so  
明显是由于libjvm.so->libjvm.so引起的（可以自己做一个实验,不要创建文件，直接执行ln -s file file ,然后cat file就会报这个错）  
知道了问题所在，就要给它引用一个真实的so文件。通过find /opt -name libjvm.so命令查找存在该文件的地方，将libjvm.so->libjvm.so指向真实文件位置。即：  
# rm -f libjvm.so  
# ln -s  具体libjvm.so文件完成路径   libjvm.so  

再次执行gmake。
结果报以下错：  
{% highlight java %}
Error: time is more than 10 years from present: 1136059200000  
java.lang.RuntimeException: time is more than 10 years from present: 1136059200000  
      at   build.tools.generatecurrencydata.GenerateCurrencyData.makeSpecialCaseEntry(GenerateCurrencyData.java:285)  
      at   build.tools.generatecurrencydata.GenerateCurrencyData.buildMainAndSpecialCaseTables(GenerateCurrencyData.java:225)  
      at   build.tools.generatecurrencydata.GenerateCurrencyData.main(GenerateCurrencyData.java:154)  
{% endhighlight %}

经过查询发现这是一个BUG.....  
下面介绍了对应补丁的出处  
https://bugs.gentoo.org/show_bug.cgi?id=534118#c3  
下面有关于问题原因探讨描述  
https://bugs.gentoo.org/show_bug.cgi?id=534118#c7  

如果不想仔细看可以直接访问补丁网址：  
http://hg.openjdk.java.net/jdk7u/jdk7u/jdk/rev/74a70385c21d#l11.1  
将其中的文件内容拷贝下来覆盖到对应文件中。  

继续gmake  

终于编译成功了！！！  
出现下面提示就说明编译完成了  
#-- Build times ----------  
Target all_product_build  
Start 2016-01-10 11:18:43  
End   2016-01-10 11:52:15  
00:00:15 corba  
00:00:21 hotspot  
00:00:11 jaxp  
00:00:20 jaxws  
00:32:15 jdk  
00:00:10 langtools  
00:33:32 TOTAL  
\-------------------------  

## 4.跑一下自己编译的虚拟机
进入目录  
/opt/openJDKroom/openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2  
下面有好几种优化级别的编译版本：  
drwxr-xr-x. 2 root root  4096 Jan  9 16:53 debug  
drwxr-xr-x. 2 root root  4096 Jan  9 16:53 fastdebug  
drwxr-xr-x. 7 root root  4096 Jan  9 16:56 generated  
drwxr-xr-x. 2 root root  4096 Jan  9 16:53 jvmg  
drwxr-xr-x. 2 root root  4096 Jan  9 16:53 optimized  
drwxr-xr-x. 3 root root 20480 Jan 10 11:19 product  
drwxr-xr-x. 2 root root  4096 Jan  9 16:53 profiled  
-rw-r--r--. 1 root root  1518 Jan  9 16:53 shared_dirs.lst  

进入到product目录中。  
这里要在env.sh配置下环境变量，指向共享库  
{% highlight shell %}
LD_LIBRARY_PATH=.:${JAVA_HOME}/jre/lib/amd64/native_threads:${JAVA_HOME}/jre/lib/amd64:/opt/openJDKroom/openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2/product  
export LD_LIBRARY_PATH
{% endhighlight %}
下面贴出我这个文件的完整内容：  
{% highlight shell %}
# Generated by /opt/openJDKroom/openjdk/hotspot/make/linux/makefiles/buildtree.make  
#: ${JAVA_HOME:=/opt/openJDKroom/jdk1.6.0_45}  
JAVA_HOME=/opt/openJDKroom/openjdk/build/linux-amd64/j2sdk-image  
CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/jre/lib/i18n.jar  
HOTSPOT_BUILD_USER="root in hotspot"  
export JAVA_HOME CLASSPATH HOTSPOT_BUILD_USER

# add  

LD_LIBRARY_PATH=.:${JAVA_HOME}/jre/lib/amd64/native_threads:${JAVA_HOME}/jre/lib/amd64:/opt/openJDKroom/openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2/product
export LD_LIBRARY_PATH  
{% endhighlight %}
下面执行：

 # source ./env.sh

 # ./gamma -version

Using java runtime at: /opt/openJDKroom/openjdk/build/linux-amd64/j2sdk-image/jre  
openjdk version "1.7.0-internal"  
OpenJDK Runtime Environment (build 1.7.0-internal-root_2016_01_10_00_16-b00)  
OpenJDK 64-Bit Server VM (build 24.0-b56, mixed mode)  

运行成功！！  

希望对想要编译openJDK的朋友有个参考。如果有疑问请提出，大家一起学习探讨。\>_<
