---
layout: post
title: Storm-集群部署
description: 在集群上部署Storm
category: "distributed"
tags: [分布式计算, 流式计算]
---

###1. 简介

&nbsp;&nbsp;&nbsp;&nbsp;[Storm](https://github.com/nathanmarz/storm)是Twitter公司的一个流式处理框架，在实时计算里面现在应该算是很流行的。
因为我在实验室里的主要工作也就是流式计算，而且做了一个类似的系统DStream，这个系统还有许多要改进的地方，目标当然是要比Storm做的更好。
Storm用clojure开发，比较难理解啊。
实际上之前还没有自己在集群上面搭建过Storm集群，导致对其理解也只是限于别人的文章的讲解，终归不能对其理解的更加深入，所以还是自己在集群上
安装一下，然后跑一些例子再说。

###2. Storm搭建过程

Storm的搭建过程主要是以下几步：安装zeromq，安装jzmq，安装java，python，unzip。
在搭建的过程中主要参考了[Storm install wiki](https://github.com/nathanmarz/storm/wiki/Setting-up-a-Storm-cluster),一篇[安装博客](http://blog.linezing.com/2013/01/how-to-install-and-deploy-storm-cluster)，和一篇[问题解答博客](http://my.oschina.net/mingdongcheng/blog/43009)。

安装环境：Ubuntu Server 12.04

####(1)安装jdk，设置JAVA_HOME
{% highlight bash %}
sudo apt-get install openjdk-7-jdk
//编辑/etc/environment，添加：
JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk-amd64"
//然后
source /etc/environment
{% endhighlight %}
####(2)安装ZooKeeper


####(3)安装zeromq
zeromq的安装需要gcc、g++等编译环境，uuid-dev等工具，以及make工具，所以可以先安装这些工具，当然也可以在下面步骤的出错信息提示后，再进行安装：
{% highlight bash%}
wget http://download.zeromq.org/zeromq-2.1.7.tar.gz
tar -xzf zeromq-2.1.7.tar.gz
cd zeromq-2.1.7
./configure
make
sudo make install
{% endhighlight %}
####(4)安装jzmq
jzmq的安装需要依赖pkg-config, autoconf, configure的过程中需要JAVA_HOME环境变量：
{% highlight bash%}
git clone https://github.com/nathanmarz/jzmq.git
cd jzmq
./autogen.sh
./configure
make
sudo make install
{% endhighlight %}
在make的时候会发生错误，参考了第三篇文章的内容终于解决了
{% highlight bash%}
make
#出现错误，提示classdist_noinst.stamp这货不存在，也没规则可以建立
#解决办法
touch src/classdist_noinst.stamp
make 
#还是会出现错误，提示没有规则去make org/zeromq/ZMQException.class
#解决办法
cd src/org/zeromq
javac *.java
cd ../../..

make
#ok了
{% endhighlight %}
###3. Storm集群的配置

Storm集群的默认参数的值可以参考网站：[Storm配置参数默认值](https://github.com/nathanmarz/storm/blob/master/conf/defaults.yaml?spm=0.0.0.0.Hh0Fp5&file=defaults.yaml)
,可以用conf/storm.yaml中的值进行覆盖。

####(1)配置Zookeeper集群
配置项storm.zookeeper.servers，内容格式如下：

	 storm.zookeeper.servers:
     - "udms-133"
     - "udms-134"
     - "udms-135"

####(2)配置nimbus，也就是master节点，内容格式如下：
	# 
 	 nimbus.host: "udms-131"
	# 

####(3)配置一个本地文件夹，用来存放storm的少量状态，jar包等，内容格式如下：
	# local dir
	 storm.local.dir: "/home/udms/storm-data"
	#
####(4)配置每个supervisor上面可以运行的workers的数量，默认是4个，要在配置文件中指定端口，默认值如下：
	 supervisor.slots.ports:
     - 6700
     - 6701
     - 6702
     - 6703

注意：将配置文件拷贝到每个节点上的conf文件夹下。


###4. Storm的启动

####(1)Nimbus的启动
在master上的storm文件夹内执行：bin/storm nimbus > /dev/null 2>&1 &

####(2)Supervisor的启动
在supervisor上的storm文件夹内执行：bin/storm supervisor > /dev/null 2>&1 &

####(3)监控页面的启动
在master节点上的storm文件夹内执行：bin/storm ui > /dev/null 2>&1 &


**注意**:可以用jps查看进程启动是否成功，在logs下的log文件中查看启动情况
