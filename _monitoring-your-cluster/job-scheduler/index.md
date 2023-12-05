---
layout: default
title: 工作调度程序
nav_order: 1
has_children: false
has_toc: false
redirect_from:
  - /job-scheduler-plugin/index/
---

# 工作调度程序

OpenSearch作业调度程序插件提供了一个框架，可用于为在群集上执行的常见任务构建计划。您可以使用作业调度程序的服务提供商界面（SPI）来定义集群管理任务的时间表，例如拍摄快照，管理数据的生命周期和运行定期作业。作业调度程序有一个清扫器，可在OpenSearch集群上聆听更新的事件，以及在运行工作时管理的调度程序。

您可以按照标准安装作业调度程序插件[OpenSearch插件安装]({{site.url}}{{site.baseurl}}/install-and-configure/install-opensearch/plugins/) 过程。例子-扩大-插件示例[工作调度程序GitHub存储库](https://github.com/opensearch-project/job-scheduler) 提供了一个完整的示例，即在构建插件时使用工作调度程序。要定义时间表，您可以构建一个实现作业调度程序库中提供的接口的插件。您可以通过指定间隔来安排作业，也可以使用Unix Cron表达式`0 12 * * ?`每天中午运行，以定义更灵活的时间表。

## 为工作调度程序构建插件

OpenSearch插件开发人员可以将作业调度程序插件扩展到计划作业以在集群上执行。您可以安排的作业包括针对原始数据运行聚合查询，将聚合数据保存到每小时的新索引中，或通过调用OpenSearch API，然后将输出发布到Webhook中，继续监视碎片分配。

有关构建使用作业调度程序插件的插件的示例，请参见作业调度程序[读书我](https://github.com/opensearch-project/job-scheduler/blob/main/README.md)。

## 定义端点

您可以通过引用插件的API端点来配置[例子](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleExtensionRestHandler.java) `SampleExtensionRestHandler.java` 文件。设置您的插件将显示的端点URL`WATCH_INDEX_URI`：

```java
public class SampleExtensionRestHandler extends BaseRestHandler {
    public static final String WATCH_INDEX_URI = "/_plugins/scheduler_sample/watch";
```

您可以通过[扩展](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobParameter.java) `ScheduledJobParameter`。您还可以定义插件使用的字段，例如`indexToWatch`，如图所示[例子](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobParameter.java) `SampleJobParameter` 文件。此作业配置将作为文档保存在您定义的索引中，如图所示[这个示例](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleExtensionPlugin.java#L54)。

## 配置参数

您可以通过引用插件的参数来配置插件的参数[例子](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobParameter.java) `SampleJobParameter.java` 文件并修改以满足您的需求：

```java
/**
 * A sample job parameter.
 * <p>
 * It adds an additional "indexToWatch" field to {@link ScheduledJobParameter}, which stores the index
 * the job runner will watch.
 */
public class SampleJobParameter implements ScheduledJobParameter {
    public static final String NAME_FIELD = "name";
    public static final String ENABLED_FILED = "enabled";
    public static final String LAST_UPDATE_TIME_FIELD = "last_update_time";
    public static final String LAST_UPDATE_TIME_FIELD_READABLE = "last_update_time_field";
    public static final String SCHEDULE_FIELD = "schedule";
    public static final String ENABLED_TIME_FILED = "enabled_time";
    public static final String ENABLED_TIME_FILED_READABLE = "enabled_time_field";
    public static final String INDEX_NAME_FIELD = "index_name_to_watch";
    public static final String LOCK_DURATION_SECONDS = "lock_duration_seconds";
    public static final String JITTER = "jitter";

    private String jobName;
    private Instant lastUpdateTime;
    private Instant enabledTime;
    private boolean isEnabled;
    private Schedule schedule;
    private String indexToWatch;
    private Long lockDurationSeconds;
    private Double jitter;
```

接下来，配置您希望插件与作业调度程序一起使用的请求参数。这些将基于您在配置插件时声明的变量。以下示例显示构建插件时设置的请求参数：

```java
public SampleJobParameter(String id, String name, String indexToWatch, Schedule schedule, Long lockDurationSeconds, Double jitter) {
        this.jobName = name;
        this.indexToWatch = indexToWatch;
        this.schedule = schedule;

        Instant now = Instant.now();
        this.isEnabled = true;
        this.enabledTime = now;
        this.lastUpdateTime = now;
        this.lockDurationSeconds = lockDurationSeconds;
        this.jitter = jitter;
    }

    @Override
    public String getName() {
        return this.jobName;
    }

    @Override
    public Instant getLastUpdateTime() {
        return this.lastUpdateTime;
    }

    @Override
    public Instant getEnabledTime() {
        return this.enabledTime;
    }

    @Override
    public Schedule getSchedule() {
        return this.schedule;
    }

    @Override
    public boolean isEnabled() {
        return this.isEnabled;
    }

    @Override
    public Long getLockDurationSeconds() {
        return this.lockDurationSeconds;
    }

    @Override public Double getJitter() {
        return jitter;
    }
```

下表描述了上一个示例中配置的请求参数。所示的所有请求参数都是必需的。

| 场地| 数据类型| 描述|
:--- | :--- | :---
| getName| 细绳| 返回工作名称。|
| getlastupdateTime| 时间单元| 返回工作最后运行的时间。|
| getEnabledtime| 时间单元| 返回启用工作的时间。|
| getSchedule| Unix Cron| 返回以UNIX CRON语法格式的工作时间表。|
| 类别| 布尔| 指示是否启用了作业。|
| getlockdurationseconds| 整数| 返回作业锁定的时间。|
| GetJitter| 整数| 返回定义的抖动值。|

您的作业使用的逻辑应由从延长的类定义`ScheduledJobRunner` 在里面`SampleJobParameter.java` 示例文件，例如`SampleJobRunner`。在运行该作业时，您可以使用一种锁定机制来防止其他节点运行相同的作业。第一的，[获得](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobRunner.java#L96) 锁。然后确保在[工作完成](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobRunner.java#L116)。

有关更多信息，请参阅工作调度程序[样品扩展](https://github.com/opensearch-project/job-scheduler/blob/main/sample-extension-plugin/src/main/java/org/opensearch/jobscheduler/sampleextension/SampleJobParameter.java) 目录中的目录[作业调度程序GitHub回购](https://github.com/opensearch-project/job-scheduler)。

## 作业调度程序集群设置

作业调度程序插件支持以下群集设置。所有设置都是动态的。要了解有关静态和动态设置的更多信息，请参阅[配置OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/configuring-opensearch/index/)。

| 环境| 数据类型| 描述|
:--- | :--- | :---
| `plugins.jobscheduler.jitter_limit` | 双倍的| 定义作业执行时间的最大延迟乘数。从同一时间开始的工作太多会导致高度的资源消耗。为了平衡负载，您可以在开始时间添加一个随机的抖动延迟。例如，如果时间间隔为10分钟，而抖动为0.6，则下一个工作运行将随机延迟0到6分钟。|
| `plugins.jobscheduler.request_timeout` | 时间单元| 背景扫描搜索超时。背景扫描是指注册作业的自动调度和执行。它发生在间隔中，并通过每个扩展插件的注册作业索引进行迭代，以搜索要执行的作业。|
| `plugins.jobscheduler.retry_count` | 整数| 用于定义指数退回策略的重试计数。退缩策略决定了大量处理器将等待多长时间，然后再进行批量操作。由于在请求时的资源限制，因此每当批量索引请求受到影响或拒绝时都会使用。对于工作调度程序插件，这会影响搜索注册的作业索引。|
| `plugins.jobscheduler.sweeper.backoff_millis` | 时间单元| 用于以毫秒为单位定义指数退回策略的初始等待期。退缩策略决定了大量处理器将等待多长时间，然后再进行批量操作。由于在请求时的资源限制，因此每当批量索引请求受到影响或拒绝时都会使用。对于工作调度程序插件，这会影响搜索注册的作业索引。|
| `plugins.jobscheduler.sweeper.page_size` | 整数| 配置用于在注册作业索引中查找作业文档的搜索请求。定义要返回的搜索命中次数。|
| `plugins.jobscheduler.sweeper.period` | 时间单元| 在执行背景扫描之前定义初始延迟期。|

