---
layout: post
title: zk-python客户端安装使用
description: 连接和使用zookeeper的python客户端
category: "distributed"
tags: [python, zookeeper]
---

##1. zk-python安装

###(1)安装zookeeper C客户端

&nbsp;&nbsp;&nbsp;&nbsp;因为ZooKeeper的python客户端依赖ZooKeeper的C客户端的实现，所以要先编译安装C客户端，如何安装zookeeper参见另外的博客；

	tar zxv -f zookeeper-3.4.3.tar.gz
	cd zookeeper-3.4.3/src/c
	./configure
	make
	sudo make install

###(2)安装zkpython包

	#下载zkpython
	wget https://pypi.python.org/packages/source/z/zkpython/zkpython-0.4.2.tar.gz
	tar zxv -f zkpython-0.4.2.tar.gz
	cd zkpython-0.4.2
	sudo python setup.py install

##2. zk-pytohn使用

###(1)import zookeeper
导入zkpython模块，如果出现： ImportError: libzookeeper_mt.so.2: cannot open shared object file: No such file or directory 错误 ，则需要添加一个环境变量：LD_LIBRATY_PATH=/usr/local/lib

###(2)建立连接：
	
	handler = zookeeper.init("172.20.1.133:2181")

###(3)创建节点：
	
	zookeeper.create(handler, "/zk_python_create_node", "mydata1", [{"perms":0x1f,"scheme":"world","id":"anyone"}], 0);

	#第一个参数：刚刚建立的连接；
	#第二个参数：创建节点的路径；
	#第三个参数：创建节点的数据；
	#第四个参数：创建节点的ACL(访问控制列表)
	#第五个参数：创建节点的类型(0表示持久化的，1表示持久化+序号，2表示瞬时的，3表示瞬时+序号)

ACL参数的意义：第一个参数是perms，代表了访问控制权限，具体值如下：

	int READ = 1<<0;
	int WRITE = 1<<1;
	int CREATE = 1<<2;
	int DELETE = 1<<3;
	int ADMIN = 1<<4;
perms的形式是一个数字，例子中的0x1f，指的是READ|WRITE|CREATE|DELETE|ADMIN

###(4)查看子节点

	zookeeper.get_children(handler,"/", None)

###(5)获取节点值及描述信息
	
	zookeeper.get(handler, "zkpython_create_node")

###(6)修改节点值

	zookeeper.set(handler, "zkpython_create_node","mydata2")

###(7)删除节点

	zookeeper.delete(handler, "/zkpython_create_node")

###(8)关闭连接
	
	zookeeper.close(handler)


##3. 参考资料
(1)[zookeeper python客户端浅析一览](http://django-china.cn/topic/143/)

(2)[在python中使用zookeeper管理你的应用集群](http://www.zlovezl.cn/articles/40/)

(3)[zookeeper客户端zkpython使用文档(一)](http://justfansty.blog.sohu.com/217953183.html)

(4)[zookeeper客户端zkpython使用文档(二)](http://justfansty.blog.sohu.com/218331818.html)