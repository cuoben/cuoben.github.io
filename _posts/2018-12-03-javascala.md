---
layout: post
title: "java和scala混合编译"
description: "jdk1.8、scala2.11"
categories: [maven,error]
tags: [编译]
redirect_from:
  - /2018/12/03/
---
# java和scala混合编译打包时，会报某某包找不到 

## 原因：打包编译时会先编译java代码再去编译scala代码，导致java代码中找不到scala的包

- 解决：
- - module一个一个打包先打scala再去打java
- - 添加maven 插件：指定先去编译scala再去编译java


~~~
<plugin>
    <groupId>net.alchim31.maven</groupId>
    <artifactId>scala-maven-plugin</artifactId>
    <version>3.2.0</version>
    <executions>
        <execution>
            <id>compile-scala</id>
            <phase>process-resources</phase>
            <goals>
                <goal>add-source</goal>
                <goal>compile</goal>
            </goals>
        </execution>
        <execution>
            <id>scala-compile-first</id>
            <phase>compile</phase>
            <goals>
                <goal>add-source</goal>
                <goal>testCompile</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <scalaVersion>2.11.8</scalaVersion>
    </configuration>
</plugin>
~~~

在process-resources处理源代码阶段中的add-source初始化源代码和compile的goal里先编译scala
