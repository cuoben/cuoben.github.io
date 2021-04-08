---
layout: post
title: "clickhouse 设置为SHA256SUM的HASH值密码"
description: "clickhouse 设置为SHA256SUM的HASH值密码"
categories: [clickhouse]
tags: [clickhouse]
redirect_from:
  - /2021/03/06/
---



# clickhouse 设置为SHA256SUM的HASH值密码

***通过公网访问clickhouse需要更改clickhouse-server目录下的config.xml，将listen_host标签改成`<listen_host>0.0.0.0</listen_host>`即通配符表示所有ip地址皆可访问，设置密码以避免安全问题。***

1. 官方不建议直接写明文密码，可以用以下命令生成密码，图中第一个输出为明文密码，第二个为通过加密的密码。

   ~~~
   PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; 
   echo -n "$PASSWORD" | sha256sum | tr -d '-';
   ~~~

![image-20210406164910512](\photo\16.png)

2. 修改users.xml文件，默认目录为/etc/clickhouse-server/users.xml，将默认的password标签改为

    `<password_sha256_hex>密码密文</password_sha256_hex>`

   



