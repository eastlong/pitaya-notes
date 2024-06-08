# ClickHouse MergeTree 原理解析

> 致谢&参考：
>
> https://blog.csdn.net/qq_40378034/article/details/120610857
>
> https://blog.csdn.net/qq_40378034/article/details/120610857



## 一、MergeTree的创建方式与存储结构

​	MergeTree在写入一批数据时，数据总会以数据片段的形式写入磁盘，且数据片段不可修改。为了避免片段过多，[ClickHouse](https://so.csdn.net/so/search?q=ClickHouse&spm=1001.2101.3001.7020)会通过后台线程，定期合并这些数据片段，属于相同分区的数据片段会被合并成一个新的片段。这种数据片段往复合并的特点，也正是合并树名称的由来。

### 1.1 MergeTree的创建方式

创建MergeTree数据表的完整语法如下所示：

```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1] [TTL expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2] [TTL expr2],
    ...
    INDEX index_name1 expr1 TYPE type1(...) GRANULARITY value1,
    INDEX index_name2 expr2 TYPE type2(...) GRANULARITY value2
) ENGINE = MergeTree()
ORDER BY expr
[PARTITION BY expr]
[PRIMARY KEY expr]
[SAMPLE BY expr]
[TTL expr [DELETE|TO DISK 'xxx'|TO VOLUME 'xxx'], ...]
[SETTINGS name=value, ...]
```

**几个重要的参数**：

1. **PARTITION BY【选填】**：分区键，用于指定数据以何种标准进行分区。分区键可以是单个列字段、元组形式的多个列字段、列表达式。如果不声明分区键，则ClickHouse会生成一个名为**all**的分区。合理使用数据分区，可以有效减少查询时数据文件的扫描范围。

2. **ORDER BY【必填】**：排序键，用于指定在一个数据片段内，数据以何种标准排序。**默认情况下主键（PRIMARY KEY）与排序键相同**。排序键可以是单个列字段（例：ORDER BY CounterID）、元组形式的多个列字段（例：ORDER BY (CounterID,EventDate)）。当使用多个列字段排序时，以ORDER BY (CounterID,EventDate)为例，在单个数据片段内，数据首先以CounterID排序，相同CounterID的数据再按EventDate排序。
3. **PRIMARY KEY【选填】**：主键，生成一级索引，加速表查询。默认情况下，主键与排序键（ORDER BY）相同，所以通常使用ORDER BY代为指定主键。一般情况下，在单个数据片段内，数据与一级索引以相同的规则升序排序。与其他数据库不同，MergeTree主键允许存在重复数据。
4. **SAMPLE BY【选填】**：抽样表达式，用于声明数据以何种标准进行采样。抽样表达式需要配合SAMPLE子查询使用
5. **SETTINGS：index_granularity【选填】**：索引粒度，默认值8192。也就是说，默认情况下每隔8192行数据才生成一条索引。
6. SETTINGS：index_granularity_bytes【选填】：在19.11版本之前，ClickHouse只支持固定大小的索引间隔（index_granularity）。在新版本中增加了自适应间隔大小的特性，即根据每一批次写入数据的体量大小，动态划分间隔大小。而数据的体量大小，由index_granularity_bytes参数控制，默认10M。
7. **SETTINGS：enable_mixed_granularity_parts【选填】**：设置是否开启自适应索引间隔的功能，默认开启。

### 1.2 MergeTree的存储结构

MergeTree的存储结构如下图所示：

<img src="../flink/images/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6YKL6YGi55qE5rWB5rWq5YmR5a6i,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center.png" alt="在这里插入图片描述" style="zoom:50%;" />

## 二、数据分区

在MergeTree中，数据是以分区目录的形式进行组织的，每个分区独立分开存储。借助这种形式，在对MergeTree进行数据查询时，可以有效跳过无用的数据文件，只使用最小的分区目录子集。

### 2.1 数据的分区规则





## 三、一级索引

MergeTree的主键使用PRIMARY KEY定义，待主键定义之后，MergeTree会依据index_granularity间隔（默认8192行），为数据表生成一级索引并保存至primary.idx文件内，索引数据按照PRIMARY KEY排序。相比使用PRIMARY KEY定义，更为常见的是通过ORDER BY指代主键。在此种情形下，PRIMARY KEY与ORDER BY定义相同，所以索引（primary.idx）和数据（.bin）会按照完全相同的规则排序。

### 稀疏索引

primary.idx文件内的一级索引采用稀疏索引实现。



稀疏索引与稠密索引的区别：

<img src="img/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6YKL6YGi55qE5rWB5rWq5YmR5a6i,size_20,color_FFFFFF,t_70,g_se,x_16" alt="在这里插入图片描述" style="zoom:50%;" />



在稠密索引中每一行索引标记都会对应到一行具体的数据记录。

而在稀疏索引中，每一行索引标记对应的是一段数据，而不是一行。由于稀疏索引占用空间小，所以primary.idx内的索引数据常驻内存。



### 索引粒度

index_granularity表示索引的粒度，默认8192。

<img src="https://img-blog.csdnimg.cn/b368a3a0dcd948b489577f3bda555d88.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA6YKL6YGi55qE5rWB5rWq5YmR5a6i,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center" alt="在这里插入图片描述" style="zoom: 67%;" />

数据以index_granularity的粒度被标记成多个小的区间，其中每个区别最多index_granularity行数据。MergeTree使用MarkRange表示一个具体的区间，并通过start和end表示其具体的范围。



### 索引数据的生成规则

由于是稀疏索引，所以MergeTree需要间隔index_granularity行数据才会生成一条索引记录，其索引值会依据声明的主键字段获取。





## 四、二级索引





