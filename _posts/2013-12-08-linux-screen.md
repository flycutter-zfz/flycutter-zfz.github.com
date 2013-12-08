---
layout: post
title: Linux screen
description: Linux screen命令的使用
category: "tools"
tags: [运维]
---

##1. 简介
在我们用putty或者telnet登陆到Linux的时候，假如要运行多个程序，而且要随时观察它们的运行情况的时候（尤其是当log直接打在标准输出的情况下），或者是我们想给自己开多个窗口，避免来回的cd，screen命令的效果就体现出来了。
<br/>
我们可以使用screen命令建立一个或者多个session，而且一个或者多个session里面还可以有多个window。

##2. 使用

1. **screen回车 或者 screen -S  session名字**，就可以建立session了; (一般我建议使用后者，毕竟自己命名的session名也好记，前者是系统会自动生成一个session名字)
   例如：`screen -S zfz`，就会生成一个pid.zfz的session了，前面那个是进程id
2. `Ctrl + a, d` 可以退出当前的session，之前的页面中去；
3. `screen -ls` 可以查看到目前系统中所有的screen的session；
4. `screen -r sessionId` 可以登录到指定的session中去；
5. 假如你要连接的session正被别人连着，可以用`screen -d -r sessionId`，连接到那个session；带来的后果是，其它连着这个session的用户从这个session上断开，然后你可以连进去；
6. `exit` 关闭session，需要先连接到要关闭的session，然后在其中执行exit命令
7. `Ctrl +a ?` 在session中显示帮助，可以查看到许多命令；下面记载的这些都可以从其中查到
8. `Ctrl +a c` 在session中创建一个新的窗口，可以在一个session中创建许多个窗口
9. `Ctrl +a x` 将当前窗口lock掉，需要密码才能重新进入
10. `Ctrl +a K` 关闭当前窗口
11. `Ctrl +a p` 回到前一个窗口
12. `Ctrl +a n` 转到下一个窗口
13. `Ctrl +a 0,1,2,..9` 切换到第几个窗口，从0开始
14. `Ctrl +a Space` 循环切换窗口
15. `Ctrl +a Ctrl +a` 在最近使用的两个窗口之间进行切换
16. `Ctrl +a w` 列出当前session的所有窗口
17. `Ctrl +a t` 显示时间

还有其它的许多命令，可以通过其help在用到的时候去查。