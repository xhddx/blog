---
author: 'xhddxiin'
title: Jmeter实现分布式压测
date: 2023-04-15
draft: false
description: "  "
thumbnail:
  url: /img/internetline.jpg
  author: JJ Ying
  authorURL: 
  origin: unsplash
  originURL: https://unsplash.com/photos/8bghKxNU1j0
---
## 1、为什么需要分布式测试？
在Jmeter中使用一个线程来模拟真实环境的一个用户，如对一个HTTP接口模拟100并发，那么在Jmeter中就需要创建100个线程来向该接口发送请求。
因为Jmeter是用Java语言开发，每创建一个线程JDK5之后的版本，JVM默认会为每个线程分配1M的堆栈内存空间（JDK5之前是256KB），可通过
-XssJVM参数调整。如果单机要模拟较大并发量的情况下，很难满足测试需求。比如对某一个服务集群模拟10万的并发，那么就需要10万个线程，
就需要98GB（100000*1/1024）左右的物理内存，很少情况会有一台超100GB内存的机器用来做测试，加上测试的需求不一。这种测试需求下，
可以将一般配置（4核8G/16G...）的多台机器组织成一个集群，在每台机器上安装并启动一个Jmeter服务，以少聚多的方式来应对大并发测试
的场景。
## 2、分布式测试原理
Jmeter分布式测试环境中有两个角色：Controller（1）和Agent（N）
Controller节点：向参与的Agent节点发送测试脚本，并聚合Agent节点的执行结果
Agent节点：接收并执行Controller节点发送过来的测试脚本，并将执行结果返回给Controller
## 3、安装部署
所有节点IP都必须安装Jmeter，且版本一致
{{< command >}}
wget http://mirror.bit.edu.cn/apache//jmeter/binaries/apache-jmeter-5.0.tgz //下载jmeter
tar -jxvf apache-jmeter-[version].tgz //解压
{{< /command >}}
## 4、设置SSL证书
* cd $JMETER_HOME/bin
* ./create-rmi-keystore.sh
* 您的名字与姓氏是什么？rmi；rmi是默认配置名称，如果不是这个名称，需要修改$JMETER_HOME/bin/jmeter.properties文件：server.rmi.ssl.keystore.alias 的值
* 最后一步密钥库口令：不要输入密码，直接回车
* 拷贝rmi_keystore.jks文件到所有节点（Controller和Agents）的bin目录下
## 5、Controller配置
* vim $JMETER_HOME/bin/jmeter.properties   把所有加入测试的Agent节点IP添加到remote_hosts配置中，多个IP之间用逗号分隔，Jmeter server 
* 默认端口号为：1099，IP后面没有加端口则使用默认端口。
{{< command >}}
remote_hosts=127.0.0.1
remote_host=192.168.0.1,192.168.0.2
{{< /command >}}
## 6、测试
* controller机器上启动jmeter
{{< command >}}
$JMETER_HOME/jmeter.sh
{{< /command >}}
* 编写好压测脚本
* 添加线程组
* jmeter -n -t ~/Desktop/dcpp-data-collect.jmx -R 192.168.0.1:1099,192.168.0.2:1099 -l ~/Desktop/dcpp.log -e -o ~/Desktop/dcpp  命令行指定远程机器运行，GUI->运行->远程运行。 不填IP则为指定所有远程机器运行
## 7、注意事项
检查防火墙是否过滤了rmi端口，默认1099.或者关闭防火墙
确保所有节点处理同一子网中