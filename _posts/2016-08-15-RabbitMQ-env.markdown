---
layout: post
title: Rabbit MQ环境搭建与集群
date: 2016-08-15
tag: Rabbit MQ
---

* TOC
{:toc}

使用生产上的版本：
Rabbit MQ:3.2.2 
erlang：R16B03

## **安装**

### **准备工作**

[下载rabbit mq](http://www.rabbitmq.com/releases/)

[下载erlang](http://www.erlang.org/download/)

### **安装erlang**

执行命令：

```bash
$ tar -zxvf otp_src_R16B03.tar.gz
$ cd otp_src_R16B03
$ ./configure
$ make
$ make install
```

测试：
命令行输入：erl
出现版本信息，则成功。

### **安装rabbitmq**

```bash
$ tar zxvf rabbitmq-server-3.2.2.tar.gz
$ cd rabbitmq-server-3.2.2
$ make
$ make install
```

在make install的时候得到一个报错：

```
echo “Put your EZs here and use rabbitmq-plugins to enable them.” > plugins/README
rm -f plugins/rabbit_common*.ez
Please set TARGET_DIR.
Please set SBIN_DIR.
Please set MAN_DIR.
Please set DOC_INSTALL_DIR.
make: *** [install_dirs] 错误 1
```

说让添加环境变量，解决方法：(rabbitmq-server-3.2.2目录下执行)

```
$ export TARGET_DIR=/home/SITAPP/rabbitmq
$ export SBIN_DIR=/home/SITAPP/rabbitmq/sbin
$ export MAN_DIR=/home/SITAPP/rabbitmq/man
$ export DOC_INSTALL_DIR=/home/SITAPP/rabbitmq/doc
```

重新make install又得到一个错误：

```
$ /bin/sh: xmlto: command not found
$ /bin/sh: line 2: xmlto: command not found
$ make: *** [docs/rabbitmqctl.1.gz] 错误 127
```

安装xmlto命令，yum install -y xmlto，yum 源不对，修改之

```
$ vi /etc/yum.repos.d/RHEL5-lan.repo
$ baseurl=http://yum.suixingpay.local/rhel/5.8/x64/Server/
```

重新make install完成。

### **配置rabbitMQ**

设置日志与消息持久化目录：

```
$ ln -s /home/SITAPP/rabbitmq/sbin/rabbitmq-server /usr/bin/rabbitmq-server
$ ln -s /home/SITAPP/rabbitmq/sbin/rabbitmq-env /usr/bin/rabbitmq-env
```

### **启动rabbitMQ**

（rabbitmq/sbin目录下）

```
$ ./ rabbitmq-server 
$ ./rabbitmq-server -detached（后台启动）
```

出现下面信息，则表示成功：

```
##########
Starting broker… completed with 0 plugins.
```

### **查看服务状态**

```
$ lsof -i:5672
```

### **关闭服务**

```bash
$ ./rabbitmqctl stop
```

### **web管理工具**

```bash
启动：./rabbitmq-plugins enable rabbitmq_management
关闭：./rabbitmq-plugins disable rabbitmq_management
出现错误
Error: {cannot_write_enabled_plugins_file,”/etc/rabbitmq/enabled_plugins”,
enoent}
创建目录： 
mkdir /etc/rabbitmq
从新启动：
 ./rabbitmq-plugins enable rabbitmq_management
出现下面信息：
The following plugins have been enabled:
mochiweb
webmachine
rabbitmq_web_dispatch
amqp_client
rabbitmq_management_agent
rabbitmq_management
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.
重启rabbitmq服务。
```

通过访问http://192.168.30.162:15672/ 用户名密码guest访问即可。

## **集群**

### **设置 Erlang Cookie**

> Erlang Cookie 文件：/var/lib/rabbitmq/.erlang.cookie or $HOME/.erlang.cookie。这里将162的该文件复制到 153，由于这个文件权限是 400，所以需要先修改153 中的该文件权限为 777：

```
$ chmod 777 XXX
```

文件存在：/root/.erlang.cookie

使用 -detached 参数运行各节点:rabbitmq-server -detached


### **组成集群**

```
$ rabbitmqctl stop_app
$ rabbitmqctl join_cluster --ram rabbit@SITAPP02
$ rabbitmqctl start_app
```

![image](\assets\images\rabbitmq_env\1.png)

查看某节点的状态
          rabbitmqctl status -n rabbit@SITAPP03
查看log tail -f /var/log/rabbit/rabbit@node1.log

### **设置镜像队列策略**
在任意一个节点上执行：

```
$ rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
```

到此集群搭建完成。