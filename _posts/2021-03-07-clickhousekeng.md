---
layout: post
title: "clickhouse配置远程连接的坑"
description: "clickhouse配置远程连接的坑"
categories: [clickhouse]
tags: [clickhouse]
redirect_from:
  - /2021/03/07/
---



# clickhouse配置远程连接的坑

**clickhouse的默认配置是不支持远程连接的只支持本机客户端连接**

1. 更改clickhouse-server目录下的config.xml，将listen_host标签改成`<listen_host>0.0.0.0</listen_host>`即通配符表示所有ip地址皆可访问

2. 更改clickhouse-server目录下的config.xml，将通过TCP协议与客户端通信的端口更改为9001，`<tcp_port>9001</tcp_port>`

3. 添加/etc/metrika.xml文件 

   ~~~
   <yandex>
           <networks>
           <ip>::/0</ip>
           </networks>
   </yandex>
   ~~~

4. 启动clicent时指定端口`--port=9001`



**问题解决**

1. ~~~
   <Error> Application: DB::Exception: Listen [0.0.0.0]:9000 failed: Poco::Exception. Code: 1000, e.code() = 98, e.displayText() = Net Exception: Address already in use: 0.0.0.0:9000 (version 20.8.3.18)
   ~~~

   即9000端口已被占用，9000为hdfs的namenode端口，在clickhouse中是和client通信的tcp协议端口，通过上述2解决。运行client时必须指定端口为9001。

2. ~~~
    <Error> Application: DB::Exception: Listen [::]:8123 failed: Poco::Exception. Code: 1000, e.code() = 0, e.displayText() = DNS error: EAI: Address family for hostname not supported (version 20.8.3.18)
   ~~~

   说明虚拟机不支持ipv6，只能对ipv4生效。在/etc/click-house/config.xml中，把<listen_host> 标签的值从::改成0.0.0.0，如上述1。



3. 本地idel测试执行抛connect time out，需向运维申请对开通公网地址暴露8123端口。