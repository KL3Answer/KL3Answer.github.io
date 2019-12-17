---
title: OpenTSDB KairosDB DolphinDB 简单性能测试
date: 2019-08-30 11:51:34
tags: 
	- 笔记
---

### [OpenTSDB](http://opentsdb.net/)

1. datapoint value 只能存储数字类型
2. 部署运维较麻烦，占用资源多
3. Java API 不完善没有官方支持，需要根据 HTTP API 自行编写
4. 性能一般
5. 文档较乱
6. 开放 HTTP API 和 Telnet API（Telnet API 只支持写数据）
7. OpenTSDB 与 HBase 版本兼容性问题较多
8. tag 特殊字符存储问题（Only the following characters are allowed: a to z, A to Z, 0 to 9, -, _, ., / or Unicode letters (as per the specification)）
9. timestamp 精度问题
10. DownSampling 与 Tags 笛卡尔积的问题
11. tag 无法直接存储中文
12. 资源占用多

### [KairosDB](https://kairosdb.github.io/)

1. 性能较OpenTSDB好
2. 开放 HTTP API 和 Telnet API（Telnet API 只支持写数据）
3. 读写一致性设置不恰当有可能会有不一致问题（可以通过设置读写一致性解决）
4. Java API 较完善
5. 使用较方便
6. DownSampling 与 Tags 笛卡尔积的问题
7. 查询请求返回数据冗余多
8. 非数值型的 datapoint 在 frist、last等 aggregator时无法获取正确的结果（在新分支中修复）
9. 支持自定义 Query 插件和数据类型



### [DolphinDB](http://dolphindb.com/)

1. 数据分区方案不够灵活
2. downsampling不够灵活,且只按时间纬度 downsampling
3. 功能多（但是各个功能的详细文档较少），适合在线分析处理
4. 闭源,收费
5. 性能好
6. 新，不久前刚出的，目前（2019.08.29）最新版是 v0.97
7. 时序数据分区不够完善
8. 使用较复杂
9. Java 数据类型转换API较麻烦
10. 使用较 KairosDB 、OpenTSDB 复杂
11. timestamp 重复不会覆盖
12. 客户端使用 TCP 进行通信
13. 不同的表类型在不同场景下各有优缺点
14. 没有 TTL
15. 客户端 API BUG 多
16. 目前写入最大批次为1024（v0..3）



### 性能测试

* 三种时序数据库都在同一台机器上进行部署，机器配置：
    * cpu:Intel(R) Xeon(R) CPU E5-2620 0 @ 2.00GHz 24核
    * memory: 125g ，clock speed 未知 ，24 插槽
    * disk:HDD，型号未知，读取速度  460.92 MB/s，写入速度约 876 MB/s

* 2000一批量插入
* OpenTSDB 和 KairosDB HTTP Connection 数为 16，DolphinDB Connection 为 16个
* 测试并发线程为8
* OpenTSDB由于特殊字符的问题，在插入output 类型 tag 之前进行了 Base64 编码
* 查询数据的时间周期跨度30天，rate 为 1h

* OpenTSDB
    * 配置
        * 单机 
        * opentsdb.conf 配置
            ```properties
            tsd.network.port = 4399
            tsd.http.staticroot = ./staticroot
            tsd.http.cachedir = /root/openTSDB_temp
            tsd.core.auto_create_metrics = true
            tsd.storage.hbase.data_table = tsdb
            tsd.storage.hbase.zk_basedir = /hbase
            tsd.storage.hbase.zk_quorum = localhost
            tsd.http.request.enable_chunked = true
            tsd.http.request.max_chunk = 1048576
            tsd.core.tag.allow_specialchars= "+=/:{}\n
            ```

        * 其他配置为默认配置,hbase（单机）、zk（单机） 也为默认配置
    * 版本 2.3.2
    * Request 使用 GZIP 压缩
* KairosDB
    * 配置
        * 单机
        * 默认配置
        * cassandra 与 KairosDB 部署在同一台机器上，也是单机默认配置
    * 版本 1.2.2-1
    * Request 使用 GZIP 压缩
* DolphinDB
    * 配置
        * 单机
        * 最大内存为32G，最多可使用8核
    * 版本 V0.97.3
    * 由于local disk 无法并发读写， distributed 表不适合横向对比，因此建 in-memory 表
        ```sql
        t=table(100000:0,`timestamp`id`name`type`dp,[timestamp,STRING,STRING,STRING,STRING])
        share t as event_00
        ```
    注： 可以使用下面建 local disk 表 测试：
    ``` sql
    dbPath = "/app/iot/dolphindb/data/U00"
    tbName = 'U00E00'
    if(existsDatabase(dbPath)){dropDatabase(dbPath)}
    db = database(dbPath,VALUE,2019.01M..2069.12M)
    t=table(100000:0,`timestamp`id`name`type`dp,[timestamp,STRING,STRING,STRING,STRING])
    db.createPartitionedTable(t,tbName,'timestamp')
    t.append!(event_00)
    db.saveTable(t,'U00E00')
    ```



dps 10,000,000

tsdb | batch put(10,000,000 records) |query range(1d,1h)| query range(30d) |query range with tag(1d,1h)|query range with tag(30d)
---- | ---| ---| ---| ---| ---| ---| ---| ---| ---| ---| ---
OpenTSDB | 51994ms(3000 per batch)| OpenTSDB 产生异常，未返回请求|OpenTSDB 产生异常，未返回请求(range :1h)|51961ms|OpenTSDB 产生异常，未返回请求(range :1h)
KairosDB | 67642ms(3000 per batch)|135911ms|单个请求超时未返回（tiemout 240s）(range :1h)|26055ms|85007ms(range :1h)
DolphinDB| 13443ms (1000 per batch)|4028ms|4209ms(range:1d)|1686ms|1336ms(range :1d)

注：查询请求时间为并发请求10次的累计时间（16线程并发）,OpenTSDB 和 kairosDB put 请求使用 Gzip 压缩,OpenTSDB,KairosDB TTL 21d,DolphinDB 查询的数据库表为 im-memory类型


dps 1000,000

tsdb | batch put(1000,000 records,1000 per batch) |query range(1d,1h)| query range(30d,1d) |query range with tag(1d,1h)|query range with tag(30d,1d)
---- | ---| ---| ---| ---| ---| ---| ---| ---| ---| ---| ---
OpenTSDB |7214ms|OpenTSDB 产生异常，未返回请求|OpenTSDB 产生异常，未返回请求|29993ms|219127ms
KairosDB |10221ms|64902ms|190634ms|25499ms|38398ms|
DolphinDB|13648ms|1052ms(in-memory)/16250ms(local-disk)|6994ms/15982ms|1201ms/12902ms|2229ms/15192ms


### 总结

* OpenTSDB
    * 强于数据写入和存储能力，但是大跨度时间范围大数据量的数据查询和分析能力弱，适合小跨度时间范围和小量数据分析场景或大时间范围但是数据经过预聚合和 rollUp 的粗粒度查询
* KairosDB
    * 适用于较长时间范围与数据量的时序数据的存储与查询分析，同时可以通过插件进行自定义数据结构与查询方式定义，可定制程度高，查询性能好
* DolphinDB
    * 分析处理数据功能强，写入查询性能强，同时提供多种模式的存储与多范式编程脚本来操作数据，缺乏时序数据的TTL、rollUp、pre-aggregate功能（但是可以通过流计算手动实现），属于偏分析性