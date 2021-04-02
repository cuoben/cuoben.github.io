---
layout: post
title: "解析maven plugin 生命周期 Lifecycle中 execution、phase、goal、mojo"
description: "解析maven plugin 生命周期 Lifecycle中 execution、phase、goal、mojo"
categories: [maven]
tags: [maven]
redirect_from:
  - /2019/08/17/
---
# 解析maven plugin 生命周期 Lifecycle中 execution、phase、goal、mojo

## **lifecycle**：生命周期，这是maven最高级别的的控制单元，它是一系列的phase组成，也就是说，一个生命周期，就是一个大任务的总称，不管它里面分成多少个子任务，反正就是运行一个lifecycle，就是交待了一个任务，运行完后，就得到了一个结果，中间的过程，是phase完成的，自己可以定义自己的lifecycle，包含自己想要的phase

-  maven 三大Lifecycle：clean、build、site
-  Default (Build) 生命周期
这是 Maven 的主要生命周期，被用于构建应用，包括下面的 23 个阶段
![image.png](/photo/a6.png)

## **phase 阶段**：可以理解为任务单元，即为上图的生命周期阶段。lifecycle是总任务，phase就是总任务分出来的一个个子任务，但是这些子任务是被规格化的，它可以同时被多个lifecycle所包含，一个lifecycle可以包含任意个phase，phase的执行是按顺序的，一个phase可以绑定很多个goal，至少为一个，没有goal的phase是没有意义的


## **goal 目标**: 这是执行任务的最小单元，它可以绑定到任意个phase中，一个phase有一个或多个goal，goal也是按顺序执行的，一个phase被执行时，绑定到phase里的goal会按绑定的时间被顺序执行，不管phase己经绑定了多少个goal，你自己定义的goal都可以继续绑到phase中


## **mojo**: lifecycle与phase与goal都是概念上的东西，mojo才是做具体事情的，可以简单理解mojo为goal的实现类，它继承于AbstractMojo，有一个execute方法，goal等的定义都是通过在mojo里定义一些注释的anotation来实现的，maven会在打包时，自动根据这些anotation生成一些xml文件，放在plugin的jar包里

