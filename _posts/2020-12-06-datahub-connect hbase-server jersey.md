---
layout: post
title: "datahub-connect和hbase-server的jersey包依赖冲突"
description: "jdk1.8、scala2.11"
categories: [maven,error]
tags: [依赖冲突]
redirect_from:
  - /2020/12/06/
---

# datahub-connect和hbase-server的jersey包依赖冲突


宽表数据写入hbase时 导入hbase-server jar包冲突报错
~~~
Caused by: java.lang.AbstractMethodError: javax.ws.rs.core.UriBuilder.uri(Ljava/lang/String;)Ljavax/ws/rs/core/UriBuilder;
	at javax.ws.rs.core.UriBuilder.fromUri(UriBuilder.java:119)
	at org.glassfish.jersey.client.JerseyWebTarget.<init>(JerseyWebTarget.java:71)
	at org.glassfish.jersey.client.JerseyClient.target(JerseyClient.java:290)
	at org.glassfish.jersey.client.JerseyClient.target(JerseyClient.java:76)
	at com.aliyun.datahub.client.http.HttpRequest.execute(HttpRequest.java:166)
	at com.aliyun.datahub.client.http.HttpRequest.executeWithRetry(HttpRequest.java:147)
	at com.aliyun.datahub.client.http.HttpRequest.fetchResponse(HttpRequest.java:136)
	at com.aliyun.datahub.client.http.HttpRequest.get(HttpRequest.java:104)
	at com.aliyun.datahub.client.impl.DatahubClientJsonImpl$8.call(DatahubClientJsonImpl.java:242)
	at com.aliyun.datahub.client.impl.DatahubClientJsonImpl$8.call(DatahubClientJsonImpl.java:238)
	at com.aliyun.datahub.client.impl.AbstractDatahubClient.callWrapper(AbstractDatahubClient.java:63)
	at com.aliyun.datahub.client.impl.DatahubClientJsonImpl.getTopic(DatahubClientJsonImpl.java:238)
	at com.alibaba.flink.connectors.datahub.datastream.source.DatahubSourceFunction.createInputSplitsForCurrentSubTask(DatahubSourceFunction.java:168
	at com.alibaba.flink.connectors.common.source.AbstractParallelSourceBase.createInitialProgress(AbstractParallelSourceBase.java:194)
	at com.alibaba.flink.connectors.common.source.AbstractParallelSourceBase.createParallelReader(AbstractParallelSourceBase.java:156)
	at com.alibaba.flink.connectors.common.source.AbstractDynamicParallelSource.createParallelReader(AbstractDynamicParallelSource.java:98)
	at com.alibaba.flink.connectors.common.source.AbstractParallelSourceBase.open(AbstractParallelSourceBase.java:141)
	at org.apache.flink.api.common.functions.util.FunctionUtils.openFunction(FunctionUtils.java:36)
	at org.apache.flink.streaming.api.operators.AbstractUdfStreamOperator.open(AbstractUdfStreamOperator.java:102)
	at org.apache.flink.streaming.runtime.tasks.OperatorChain.initializeStateAndOpenOperators(OperatorChain.java:291)
	at org.apache.flink.streaming.runtime.tasks.StreamTask.lambda$beforeInvoke$0(StreamTask.java:479)
	at org.apache.flink.streaming.runtime.tasks.StreamTaskActionExecutor$SynchronizedStreamTaskActionExecutor.runThrowing(StreamTaskActionExecutor.java:92)
	at org.apache.flink.streaming.runtime.tasks.StreamTask.beforeInvoke(StreamTask.java:475)
	at org.apache.flink.streaming.runtime.tasks.StreamTask.invoke(StreamTask.java:528)
	at org.apache.flink.runtime.taskmanager.Task.doRun(Task.java:721)
	at org.apache.flink.runtime.taskmanager.Task.run(Task.java:546)
	at java.lang.Thread.run(Thread.java:748)
~~~
- 首先通过报错定位到冲突的jar为jersey 通过mvn dependency:tree  -Dinclude=jersey 找到jersey的详细依赖关系
发现是datahub-connector和habse-server同时依赖jersey
![image.png](/photo/a1.png)
![image.png](/photo/a2.png)
- 问题找到了就找解决办法：
确定jersey的版本
![image.png](/photo/a3.png)

- 又报另一个错

com.aliyun.datahub.client.exception.DatahubClientException: [httpStatus:5001, requestId:null, errorCode:null, errorMessage:javax.ws.rs.core.Response$Status$Family.familyOf(I)Ljavax/ws/rs/core/Respons.]

- 在maventree中发现javax.ws.rs又依赖于jersey
![image.png](/photo/a4.png)

- 于是在引入的jersey中exclusion了javax.ws.rs
![image.png](/photo/a5.png)

 - 问题解决 网上无相关报错 花了一天时间
