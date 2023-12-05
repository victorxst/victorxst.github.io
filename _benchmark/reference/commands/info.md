---
layout: default
title: 信息
nav_order: 75
parent: 命令参考
grand_parent: OpenSearch基准参考
redirect_from: /benchmark/commands/info/
---

# 信息

这`info` 命令打印有关OpenSearch基准组件的详细信息。

## 用法

以下示例返回有关名为工作负载的信息`nyc_taxis`：

```
opensearch-benchmark info --workload=nyc_taxis
```

OpenSearch Benchmark返回有关工作量的信息，如以下示例响应中所示：

```
   ____                  _____                      __       ____                  __                         __
  / __ \____  ___  ____ / ___/___  ____ ___________/ /_     / __ )___  ____  _____/ /_  ____ ___  ____ ______/ /__
 / / / / __ \/ _ \/ __ \\__ \/ _ \/ __ `/ ___/ ___/ __ \   / __  / _ \/ __ \/ ___/ __ \/ __ `__ \/ __ `/ ___/ //_/
/ /_/ / /_/ /  __/ / / /__/ /  __/ /_/ / /  / /__/ / / /  / /_/ /  __/ / / / /__/ / / / / / / / / /_/ / /  / ,<
\____/ .___/\___/_/ /_/____/\___/\__,_/_/   \___/_/ /_/  /_____/\___/_/ /_/\___/_/ /_/_/ /_/ /_/\__,_/_/  /_/|_|
    /_/

Showing details for workload [nyc_taxis]:

* Description: Taxi rides in New York in 2015
* Documents: 165,346,692
* Compressed Size: 4.5 GB
* Uncompressed Size: 74.3 GB

===================================
TestProcedure [searchable-snapshot]
===================================

Measuring performance for Searchable Snapshot feature. Based on the default test procedure 'append-no-conflicts'.

Schedule: 
----------

1. delete-index
2. create-index 
3. check-cluster-health
4. index (8 clients)
5. refresh-after-index
6. force-merge
7. refresh-after-force-merge
8. wait-until-merges-finish
9. create-snapshot-repository
10. delete-snapshot
11. create-snapshot
12. wait-for-snapshot-creation
13. delete-local-index
14. restore-snapshot
15. default 
16. range
17. distance_amount_agg
18. autohisto_agg
19. date_histogram_agg

====================================================
TestProcedure [append-no-conflicts] (run by default) 
====================================================

Indexes the entire document corpus using a setup that will lead to a larger indexing throughput than the default settings and produce a smaller index (higher compression rate). Document IDs are unique, so all index operations are append only. After that, a couple of queries are run. 

Schedule:
----------

1. delete-index
2. create-index
3. check-cluster-health
4. index (8 clients)
5. refresh-after-index
6. force-merge
7. refresh-after-force-merge
8. wait-until-merges-finish
9. default
10. range
11. distance_amount_agg
12. autohisto_agg
13. date_histogram_agg

==============================================
TestProcedure [append-no-conflicts-index-only]
==============================================

Indexes the whole document corpus using a setup that will lead to a larger indexing throughput than the default settings and produce a smaller index (higher compression rate). Document ids are unique so all index operations are append only.

Schedule:
----------

1. delete-index
2. create-index
3. check-cluster-health
4. index (8 clients)
5. refresh-after-index
6. force-merge
7. refresh-after-force-merge
8. wait-until-merges-finish

=====================================================
TestProcedure [append-sorted-no-conflicts-index-only]
=====================================================

Indexes the whole document corpus in an index sorted by pickup_datetime field in descending order (most recent first) and using a setup that will lead to a larger indexing throughput than the default settings and produce a smaller index (higher compression rate). Document ids are unique so all index operations are append only.

Schedule:
----------

1. delete-index
2. create-index
3. check-cluster-health
4. index (8 clients)
5. refresh-after-index
6. force-merge
7. refresh-after-force-merge
8. wait-until-merges-finish

======================
TestProcedure [update]
======================

Schedule:
----------

1. delete-index
2. create-index
3. check-cluster-health
4. update (8 clients)
5. refresh-after-index
6. force-merge
7. refresh-after-force-merge
8. wait-until-merges-finish


-------------------------------
[INFO] SUCCESS (took 2 seconds)
-------------------------------
```

## 选项

您可以使用以下选项与`info` 命令：


- `--workload-repository`：从OpenSearch基准加载工作负载的位置定义存储库。
- `--workload-path`：定义下载或自定义工作负载的路径。
- `--workload-revision`：定义了OpenSearch基准应使用的工作负载源树的特定修订。
- `--workload`：定义要根据工作负载的名称使用的工作负载。您可以使用`opensearch-benchmark list workloads`。
- `--test-procedure`：定义使用的测试过程。您可以使用`opensearch-benchmark list test_procedures`。
- `--include-tasks`：定义逗号-分开运行的测试过程列表。默认情况下，运行测试过程中列出的所有任务均已运行。
- `--exclude-tasks`：定义逗号-分开的测试过程任务列表不运行。

