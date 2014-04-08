---
layout: post
title: "Linux Ethtool"
description: "The using of ethtool"
category: ""
tags: [Linux Internet]
---

##1.简介
ethtool可以用来查看Linux系统的网络端口情况。在一次部署过程中，发现两台服务器之间拷贝文件非常慢，大概在7MB/s的速度。
工程师推测是有一台服务器的网卡为百兆，经过查看果然如此。
网线查到网口上会有两个灯在闪，如果两个都是绿灯，则说明是百兆；如果一个黄灯一个绿灯，则说明是千兆。

使用ethtool可以查看一个网口的状态，命令为：
`ethtool eth0`

在服务器上的运行结果如下：
![ethtool运行结果](/assets/images/ethtool.png)

从图上可以清楚的看到网口的状态为1000MB/s
