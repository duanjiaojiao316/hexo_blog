---
title: 关于YARN——基础知识
categories:
  - null
tags:
  - null
translate_title: about-yarnbasic-knowledge
date: 2021-03-09 21:56:09
---


### 1、什么是YARN

YARN是Hadoop的集群资源管理系统，在Hadoop 2被引入，为了盖上MapReduce的实现。YARN提供请求和使用资源的API，这些API向用户隐藏了资源管理的细节。一些分布式计算框架MapReduce、spark运行在YARN和存储层HDFS/HBase上。

集群层次框架：

| Upper Application | Pig/Hive/Crunch                |
| ----------------- | ------------------------------ |
| Application       | `MapReduce`/`Spark`/`Tez` /... |
| Compute           | YARN                           |
| Storage           | HDFS and HBase                 |

YARN通过两类长期运行的守护进程提供自己的服务

> 资源管理器：管理集群上资源使用
>
> 节点管理器：运行在集群的所有节点上，启动和监控容器

container 容器：执行job进程，容器有资源限制(内存，CPU等)，具体容器取决于YARN配置

### 2、YARN的运行机制：

<img src="%E5%85%B3%E4%BA%8EYARN%E2%80%94%E2%80%94%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/201806211939097.jpg" alt="img" style="zoom:67%;" />

1) 首先客户端请求RM，运行一个application master

2) RM找到可以在容器中启动application master的NM，在NM启动容器，运行application master

3) 容器通过心跳机制向RM请求运行资源(内存和CPU)

4) application master运行起来之后需要做什么依赖于客户端传递的应用

a. 简单地运算后直接返回结果给客户端

b. 请求更多容器进行分布式计算

#### YARN通信手段：

YARN不提供任何手段用于应用各部分(客户端/application master/进程)通信，使用**具体应用的远程通信机制(如Hadoop的RPC层)**来向客户端返回状态和结果

#### YARN资源请求的灵活性：

1) 指定每个容器的计算机资源数量(内存和CPU)

2) 容器的本地限制要求，可以申请指定节点，机架或集群中任意位置(包含集群外)的容器

3) 可以在运行中的任意时刻请求资源，可以一开始请求所有资源(Spark)；也可以动态申请，随着应用程序的运行而动态地请求(MapReduce先请求Map任务资源，后请求Reduce任务资源）

#### 应用生命期-三种模型

这里的应用指的是application master，用来处理job的YARN的应用进程

1) 一个用户作业对应一个application master（MapReduce）

2) 作业的每个工作流或者每个用户对话对应一个application master

效率高，容器可以在作业之间重用，可能缓存中间数据，Spark使用这种模型

3) 多个用户共享一个长期运行的应用

这种application master作为协调者身份运行，一个always on的application master，则无需启动新的application master，低开销，低响应延迟

### 3、YARN的调度

YARN调度器根据既定策略为应用分配资源，由于调度难题没有所谓”最好“的策略，YARN提供多种调度器和可配置策略。

#### YARN三种调度器

1）FIFO调度器（FIFO Scheduler）：应用放置在一个队列中，按照作业提交的顺序运行

2）容量调度器（Capacity Scheduler）：独立的专门队列保证小作业也可以提交后就启动，队列容量是专门保留的，以整个集群的利用率为代价，与FIFO比，大作业执行的时间要长

3）公平调度器（Fair Scheduler）：不需要预留一定的资源，调度器会在所有的运行的作业之间动态平衡资源，第一个（大）作业启动时，他是唯一运行的作业所以获得集群中的所有资源，当第二个作业启动时，它被分配到集群的一半资源，每个作业都能公平共享资源。

第二个作业从启动到获得资源会有一定的时间滞后，它必须等第一个大作业使用的容器用完并释放资源，小作业结束并且不再使用资源之后，大作业将回去再次使用全部的资源。最终的效果：既得到了较高的集群利用率，又能保证小作业及时完成

<img src="关于YARN——基础知识/20180621201207617" alt="img" style="zoom: 80%;" />

------

#### 容量调度器配置

容量调度器允许多个组织共享一个Hadoop集群，每个组织可以分配到全部集群资源的一部分。每个组织有一个专门的队列，每个队列被配置使用集群的一定资源。队列也可以按照层次进一步划分，这样每个组织内部的不同用户能共享该组织队列的全部资源，在一个队列中采取FIFO调度策略。

如果队列中有多个作业，队列资源不够，此时如果其他队列有空闲的资源，容量调度器会把空闲的资源分配给队列中的作业，会超出队列容量。这种成为弹性队列。

正常操作时不会出现抢占队列资源，如果队列资源不够用，需要等其他队列释放容器资源。

为队列设置自最大容量限制，这样队列就不会过多的侵占其他队列的容量，这种方法以牺牲弹性队列为代价防止过多侵占。

假设一个队列层次

![image-20210310213545443](关于YARN——基础知识/image-20210310213545443.png)

root队列下，prod和dev队列分别占40%和60%的容量。配置文件如下

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.scheduler.capacity.root.queues</name>
        <value>prod,dev</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.dev.queues</name>
        <value>eng,science</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.prod.capacity</name>
        <value>40</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.dev.capacity</name>
        <value>60</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.dev.maximum-capacity</name>
        <value>75</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.dev.eng.capacity</name>
        <value>50</value>
    </property>
    <property>
        <name>yarn.scheduler.capacity.root.dev.science.capacity</name>
        <value>50</value>
    </property>
</configuration>
```

`dev`队列最大容量设置为75%，其实`prod`队列是空闲的，dev队列也不会占用全部的集群资源。`prod`队列使用的资源占比总能达到25%。由于没有对`eng`和`science`队列的最大容量进行限制，其中一个队列中的作业可能会占用`dev`队列中的所有资源（相当于全部集群资源的75%），而`prod`队列可以占用集群的全部资源。



通过`mapreduce.job.queuename`属性设置应用的队列，不指定应用将放在名为`“default”`的默认队列中。

在容量调度器，队列的名字是队列层次的最后一部分，完整队列层次名不会被识别。

`prod`和`dev`是合法的队列名，`root.dev.eng`作为队列名无效。

------

#### 公平调度器配置

假如有A和B两个用户，分别拥有队列`queueA`和队列`queueB`，A启动一个作业，B此时没有需求，A会分到队列的全部资源，当A还在运行，B启动第一个作业，将占用一半资源，此时如果B启动第二个作业，其他的作业都还在运行，第二个作业将和B共享资源，获取队列B的一半资源。

 ![image-20210310221541899](关于YARN——基础知识/image-20210310221541899.png)

##### 1、启动公平调度器

公平调度器的使用由属性` yarn.resourcemanager.scheduler.class` 的设置所决定。默认是使用 Capacity Scheduler (尽管在一些Hadoop分布式项目， 如CDH中是默认使用 `Fair Scheduler`)，如果要使用 `Fair Scheduler`，需要修改`yarn-site.xml` 文件。

```xml
<property>
     <name>yarn.resourcemanager.scheduler.class</name>
     <value>
         org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler
    </value>
</property>
```

##### 2、队列配置

通过一个名为` fair-scheduler.xml `的分配文件对 `Fair Scheduler` 进行配置， 该文件位于类路径下。(可以通过设置属性`yarn.scheduler.fair.allocation.file`来修改文件名) 。当没有该分配文件时，`Fair Scheduler `的工作策略同先前所描述的一样：每个应用放置在一个以用户名命名的队列中，队列是在用户提交第一个应用时动态创建的。

通过分配文件可以为每个队列进行配置。这样可以对 Fair Scheduler 支持的层次队列进行配置。 例如，可以像为`Capacity Scheduler `所做的那样，使用分配文件定义 prod 和 dev。

```xml
<?xml version="1.0"?>
<allocations>
  <defaultQueueSchedulingPolicy>fair</defaultQueueSchedulingPolicy>

  <queue name="prod">
    <weight>40</weight>
    <schedulingPolicy>fifo</schedulingPolicy>
  </queue>

  <queue name="dev">
    <weight>60</weight>
    <queue name="eng" /> 
    <queue name="science" />
  </queue>

  <queuePlacementPolicy>
    <rule name="specified" create="false" />
    <rule name="primaryGroup" create="false" />
    <rule name="default" queue="dev.eng" />
  </queuePlacementPolicy>
</allocations>
```

3、队列放置

1) 使用基于规则的系统确定job队列放置，匹配对应的用户队列直到使用default队列

2) 直接就使用default，所有job公平分配

第一条规则， specified， 表示把应用放进所指明的队列中， 如果没有指明，或如果所指明的队列不存在，则规则不匹配，继续尝试下一条规则。

primary Group规则会试着把应用放在以用户的主Unix组名命名的队列中，如果没有这样的队列， 则继续尝试下一条规则而不是创建队列。

Defahult规则是 一条兜底规则， 当前述规则都不匹配时， 将启用该条规则， 把应用放进 `dev.eng` 队列中。