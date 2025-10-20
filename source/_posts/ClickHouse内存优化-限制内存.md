---
title: ClickHouse内存优化-限制内存
date: 2025-09-18 10:18:14
tags: 
  - Docker
  - ClickHouse
top: false
categories: 
  - 内存优化
  - 困难解决
img: /medias/featureimages/9.jpg

---

# ClickHouse内存优化
**问题现象容器化部署的ClickHouse在运行较长时间后，运行内存会一直增长，直到到达服务器内存峰值，导致服务器运行其他服务效率变慢**
## 通过修改clickhouse的配置文件，设定运行内存最大值
``` bash
#mark_cache_size参数用于配置 ClickHouse 中 ​​标记缓存(Mark Cache)​​ 的内存大小上限，这是 MergeTree 引擎家族（包括 ReplicatedMergeTree 等）的关键性能优化机制。（单位：字节）
#clickhouse/config.xml文件中的 mark_cache_size 配置 修改为  524288000
<yandex>
    <mark_cache_size>524288000</mark_cache_size> <!-- 500MB -->
</yandex>
#或临时设置
SET mark_cache_size = 8589934592; -- 8GB


#max_block_size参数用于控制查询处理时单个数据块(block)的最大行数，影响内存使用和查询处理效率，较大的值可以提高吞吐量，但会增加内存消耗，较小的值可以减少内存使用，但可能降低性能，默认值：65536
<max_block_size>8192</max_block_size>
#nput_format_parallel_parsing参数用于控制是否启用输入数据的并行解析
#启用(true)时，ClickHouse 会使用多线程并行解析输入数据
#对于大文件导入(如CSV, JSON等格式)可显著提高性能
#默认值：true = 1  false = 0
<input_format_parallel_parsing>0</input_format_parallel_parsing>
#output_format_parallel_formatting参数用于控制是否启用输出数据的并行格式化
#启用(true)时，ClickHouse 会使用多线程并行格式化输出数据
#对于大结果集导出可提高性能
#默认值：true = 1  false = 0
<output_format_parallel_formatting>0</output_format_parallel_formatting>
#汇总配置
<yandex>
    <profiles>
        <default>
            <max_block_size>65536</max_block_size>
            <input_format_parallel_parsing>true</input_format_parallel_parsing>
            <output_format_parallel_formatting>true</output_format_parallel_formatting>
        </default>
    </profiles>
</yandex>


```
​​max_block_size​​

增大可提高复杂查询性能，但会增加内存使用

典型调优范围：8192-131072

监控内存使用情况调整
​
​并行参数​​

在多核服务器上效果显著

可减少I/O等待时间

对小数据量操作可能增加开销