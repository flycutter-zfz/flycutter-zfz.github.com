---
layout: post
title: Linux cron job
description: Linux定时任务
category: "tools"
tags: [运维]
---

##1. 简介
cron job 是Linux定时任务的服务。可以定时：时、分、月、日、星期进行定时任务和计划任务

##2. 语法
`crontab [-u username] [-l] [-r] [-e]`
1. -u: 指定crontab job的用户
2. -l:  列出当前crontab的job
3. -e: 编辑crontab job，默认使用nano编辑器，可以添加环境变量EDITOR来改变编辑器，比如用vim，export EDITOR="/usr/bin/vim";
4. -r: 删除当前crontab job

##3. 使用
使用crontab file来编辑cron job是个不错的选择<br/>
**文件格式：**
每行对应一个cron job；<br/>
每一行分为6个部分，用空格分开，同一个部分有需要的话用逗号分开；<br/>
`minute   hour   day_of_month   month   weekday     command`<br/>
前5个域中，使用`*`，表示所有的时间点；<br/>
minute: 0-59<br/>
hour: 0-23, 0代表零点；<br/>
day_of_month: 1-31;<br/>
weekday: 0-6, 0代表星期天，1-6代表周一到周六;<br/>
command：需要执行的脚本或者命令；<br/>

##4. 举例
test_cron.sh内容：
	#!/bin/bash
	echo `date` >> time.txt


编辑mycrontab文件：
`* * * * * /home/udms/test_cron.sh`
之后执行：crontab mycrontab 通知系统执行mycrontab定义的cron job

也可以直接执行：crontab -e编辑，直接输入 `* * * * * /home/udms/test_cron.sh`


其它例子：
1. `0 6 * * * command` ，表示每天早上6点钟执行命令；
2. `0 */2 8 * * command`，表示每两个小时执行一次命令；
3. `0 11 4 * 1-3 command`，表示每个月的4号，和每周的周一到周三的早上11点执行命令；
4. `0 4 1 1 * command`， 表示1月1号早上4点执行命令；

`/`：表示频率；<br/>
`-`：表示范围；<br/>
`#`：注释符号；<br/>
