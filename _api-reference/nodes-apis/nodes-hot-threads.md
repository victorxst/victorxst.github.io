---
layout: default
title: 节点热线
parent: 节点API
nav_order: 30
---

# 节点热线
**引入1.0**
{: .label .label-purple }

节点热线程端点提供有关所选群集节点繁忙的JVM线程的信息。它提供了每个节点活动的独特视图。

#### 例子

```json
GET /_nodes/hot_threads
```
{% include copy-curl.html %}

## 路径和HTTP方法

```json
GET /_nodes/hot_threads
GET /_nodes/<nodeId>/hot_threads
```

## 路径参数

您可以在请求中包含以下可选路径参数。

范围| 类型| 描述
:--- | :--- | :---
nodeid| 细绳| 逗号-用于过滤结果的节点ID的分离列表。支持[节点过滤器]({{site.url}}{{site.baseurl}}/api-reference/nodes-apis/index/#node-filters)。默认为`_all`。

## 查询参数

您可以在请求中包含以下查询参数。所有查询参数都是可选的。

范围| 类型| 描述
:--- | :---| :---
快照| 整数| 线程堆栈的样本数量。默认为`10`。
间隔| 时间| 连续样品之间的间隔。默认为`500ms`。
线程| 整数| 返回有关信息的最繁忙线程的数量。默认为`3`。
ignore_idle_threads| 布尔| 不要显示已知空闲状态的线程，例如在插座上等待或从空任务队列中拉出的线程。默认为`true`。
类型| 细绳| 支持的线程类型是`cpu`，`wait`， 或者`block`。默认为`cpu`。
暂停| 时间| 设置节点响应的时间限制。默认值是`30s`。

#### 示例请求

```json
GET /_nodes/hot_threads
```
{% include copy-curl.html %}

#### 示例响应

```bash
::: {opensearch}{F-ByTQzVQ3GQeYzQJArJGQ}{GxbcLdCATPWggOuQHJAoCw}{127.0.0.1}{127.0.0.1:9300}{dimr}{shard_indexing_pressure_enabled=true}
   Hot threads at 2022-09-29T19:46:44.533Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
   
    0.1% (455.5micros out of 500ms) cpu usage by thread 'ScheduledMetricCollectorsExecutor'
     10/10 snapshots sharing following 2 elements
       java.base@17.0.4/java.lang.Thread.sleep(Native Method)
       org.opensearch.performanceanalyzer.collectors.ScheduledMetricCollectorsExecutor.run(ScheduledMetricCollectorsExecutor.java:100)
```

## 回复

与大多数OpenSearch API响应不同，此响应是文本格式。

它由响应中包含的每个群集节点的一个部分组成。

每个部分以包含以下段的一行开始：

线段| 描述
：--- |：-------
<code> :::＆nbsp; </code>| 线启动（一个独特的视觉符号）。
`{global-eu-35}` | 节点名称。
`{uFPbKLDOTlOmdnwUlKW8sw}` | nodeid。
`{OAM8OT5CQAyasWuIDeVyUA}` | 临时性。
`{global-eu-35.local}` | 主机名。
`{[gdv2:a284:2acv:5fa6:0:3a2:7260:74cf]:9300}` | 主机地址。
`{dimr}` | 节点角色（d = data，i = ingest，m = cluster＆nbsp; manager，r =远程＆nbsp; cluster＆nbsp; client）。
`{zone=west-a2, shard_indexing_pressure_enabled=true}` | 节点属性。

然后提供有关所选类型线程的信息。

```bash
::: {global-eu-35}{uFPbKLDOTlOmdnwUlKW8sw}{OAM8OT5CQAyasWuIDeVyUA}{global-eu-35.local}{[gdv2:a284:2acv:5fa6:0:3a2:7260:74cf]:9300}{dimr}{zone=west-a2, shard_indexing_pressure_enabled=true}
   Hot threads at 2022-04-01T15:15:27.658Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
   
    0.1% (645micros out of 500ms) cpu usage by thread 'opensearch[global-eu-35][transport_worker][T#7]'
     4/10 snapshots sharing following 3 elements
       io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:986)
       io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
       java.base@11.0.14.1/java.lang.Thread.run(Thread.java:829)
::: {global-eu-62}{4knOxAdERlOB19zLQIT1bQ}{HJuZs2HiQ_-8Elj0Fvi_1g}{global-eu-62.local}{[gdv2:a284:2acv:5fa6:0:3a2:bba6:fe3f]:9300}{dimr}{zone=west-a2, shard_indexing_pressure_enabled=true}
   Hot threads at 2022-04-01T15:15:27.659Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
      
   18.7% (93.4ms out of 500ms) cpu usage by thread 'opensearch[global-eu-62][transport_worker][T#3]'
     6/10 snapshots sharing following 3 elements
       io.netty.util.concurrent.SingleThreadEventExecutor$4.run(SingleThreadEventExecutor.java:986)
       io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74)
       java.base@11.0.14.1/java.lang.Thread.run(Thread.java:829)
::: {global-eu-44}{8WW3hrkcTwGvgah_L8D_jw}{Sok7spHISFyol0jFV6i0kw}{global-eu-44.local}{[gdv2:a284:2acv:5fa6:0:3a2:9120:e79e]:9300}{dimr}{zone=west-a2, shard_indexing_pressure_enabled=true}
   Hot threads at 2022-04-01T15:15:27.659Z, interval=500ms, busiestThreads=3, ignoreIdleThreads=true:
   
   42.6% (212.7ms out of 500ms) cpu usage by thread 'opensearch[global-eu-44][write][T#5]'
     2/10 snapshots sharing following 43 elements
       java.base@11.0.14.1/sun.nio.ch.IOUtil.write1(Native Method)
       java.base@11.0.14.1/sun.nio.ch.EPollSelectorImpl.wakeup(EPollSelectorImpl.java:254)
       io.netty.channel.nio.NioEventLoop.wakeup(NioEventLoop.java:787)
       io.netty.util.concurrent.SingleThreadEventExecutor.execute(SingleThreadEventExecutor.java:846)
       io.netty.util.concurrent.SingleThreadEventExecutor.execute(SingleThreadEventExecutor.java:815)
       io.netty.channel.AbstractChannelHandlerContext.safeExecute(AbstractChannelHandlerContext.java:989)
       io.netty.channel.AbstractChannelHandlerContext.write(AbstractChannelHandlerContext.java:796)
       io.netty.channel.AbstractChannelHandlerContext.writeAndFlush(AbstractChannelHandlerContext.java:758)
       io.netty.channel.DefaultChannelPipeline.writeAndFlush(DefaultChannelPipeline.java:1020)
       io.netty.channel.AbstractChannel.writeAndFlush(AbstractChannel.java:311)
       org.opensearch.transport.netty4.Netty4TcpChannel.sendMessage(Netty4TcpChannel.java:159)
       app//org.opensearch.transport.OutboundHan...
```

## 所需的权限

如果使用安全插件，请确保设置以下权限：`cluster:monitor/nodes/hot_threads`。

