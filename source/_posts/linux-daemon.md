---
title: linux守护进程
comments: true
tags:
  - linux
  - nodejs
  - tmux
date: 2019-02-13 16:05:29
---

###### 介绍

["守护进程"](http://baike.baidu.com/view/53123.htm)（daemon）就是一类在后台运行的特殊进程，用于执行特定的系统任务，很多守护进程在系统引导时候启动并一直运行到系统关闭，也有一些在需要时候启动，完成任务后自动结束。

##### 问题
一个简单的node server.js 一旦退出命令行窗口，这个应用就一起退出，无法访问了。那么如何才能让他一直运行呢？有以下办法：

##### 办法一 &

> $ node server.js &
> 只要在命令的尾部加上符号`&`，启动的进程就会成为"后台任务"。

##### 办法二  sighup信号 disown

>todo

##### 办法三 nohup命令

> nohup node server.js $

nodup命令做了三件事:
1.阻止sighup信号发到这个进程
2.关闭标准输入
3.重定向标准输出和标准错误到文件nohup.out

##### 办法四 screen命令和tmux命令

终端复用器 terminal multiplexer ：在同一个终端里，管理多个session典型的就是screen和tmux

screen用法

```bash
#新建一个session
$ screen
$ node server.js

$ screen -S name

# 切回指定 session
$ screen -r name
$ screen -r pid_number

# 列出所有 session
$ screen -ls

```

tmux用法

  ```bash
   #Debian系和Ubuntu：
  sudo apt install tmux
  #Redhat系包括CentOS
  yum install -y tmux
  #在macOS上安装Tmux
  brew install tmux
  #启动新session：
  $ tmux [new -s 会话名 -n 窗口名]
  
  #恢复session：
  $ tmux at [-t 会话名]
  
  #列出所有sessions：
  $ tmux ls
  
  #重新命名会话sessions：
  $ tmux rename -t oldName newName
  
  #关闭session：
  $ tmux kill-session -t 会话名
  
  #关闭整个tmux服务器：
  $ tmux kill-server
  ```

##### 办法五 Node工具

针对node有一些专门的启动工具：：[forever](https://github.com/foreverjs/forever)，[nodemon](http://nodemon.io/) 和 [pm2](http://pm2.keymetrics.io/)。
forever 保证进程退出时，应用会自动重启。

```bash
# 作为前台任务启动
$ forever server.js

# 作为服务进程启动 
$ forever start app.js

# 停止服务进程
$ forever stop Id

# 重启服务进程
$ forever restart Id

# 监视当前目录的文件变动，一有变动就重启
$ forever -w server.js

# -m 参数指定最多重启次数
$ forever -m 5 server.js 

# 列出所有进程
$ forever list
```

`nodemon`一般只在开发时使用，它最大的长处在于 watch 功能，一旦文件发生变化，就自动重启进程。

> ```bash
> # 默认监视当前目录的文件变化
> $ nodemon server.js
> 
> ＃ 监视指定文件的变化   
> $ nodemon --watch app --watch libs server.js  
> ```

pm2 的功能最强大，除了重启进程以外，还能实时收集日志和监控。

> ```bash
> # 启动应用
> $ pm2 start app.js
> 
> # 指定同时起多少个进程（由CPU核心数决定），组成一个集群
> $ pm2 start app.js -i max
> 
> # 列出所有任务
> $ pm2 list
> 
> # 停止指定任务
> $ pm2 stop 0
> 
> ＃ 重启指定任务
> $ pm2 restart 0
> 
> # 删除指定任务
> $ pm2 delete 0
> 
> # 保存当前的所有任务，以后可以恢复
> $ pm2 save
> 
> # 列出每个进程的统计数据
> $ pm2 monit
> 
> # 查看所有日志
> $ pm2 logs
> 
> # 导出数据
> $ pm2 dump
> 
> # 重启所有进程
> $ pm2 kill
> $ pm2 resurect
> 
> # 启动web界面 http://localhost:9615
> $ pm2 web
> ```

##### 办法六 Systemd

为系统的启动和管理提供一套完整的解决方案，d就是守护daemon的意思，它守护整个系统，替代initd
优点是功能强大 缺点是体系庞大太复杂。

systemctl是主命令，用于管理系统

```bash
# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# CPU停止工作
$ sudo systemctl halt

# 暂停系统
$ sudo systemctl suspend

# 让系统进入冬眠状态
$ sudo systemctl hibernate

# 让系统进入交互式休眠状态
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue
```

`systemd-analyze`命令用于查看启动耗时

```bash
# 查看启动耗时
$ systemd-analyze                                                                                       

# 查看每个服务的启动耗时
$ systemd-analyze blame

# 显示瀑布状的启动过程流
$ systemd-analyze critical-chain

# 显示指定服务的启动流
$ systemd-analyze critical-chain atd.service

```

`hostnamectl`命令用于查看当前主机的信息。

```bash
# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7
```

unit管理

 ```bash
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
 ```





###### 参考网站

[Tmux入门介绍](http://blog.jobbole.com/87278/)
[Linux 守护进程的启动方法](https://www.cnblogs.com/0616--ataozhijia/p/8037887.html)
[Systemd命令](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)