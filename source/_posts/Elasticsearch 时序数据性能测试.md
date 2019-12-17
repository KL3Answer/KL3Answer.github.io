---
title: Elasticsearch  时序数据性能测试
date: 2019-09-02 18:32:01
tags: 
	- 笔记
---

### Elasticsearch

1. 2 billion docs per shard at most
2. 手动清除过期数据(index)
3. 会产生大量的index 和 shard
4. 记录 index 分派策略为 INDEX_PATTERN_NAME{pk}_{yyyyMMdd}，其中pk为当前记录所属的产品的productKey，时间为当前的记录中的日期


### 测试结果

* ES 版本 6.8

* ES 设置

    * 开启自动创建 index
    ```
    PUT _cluster/settings
    {
        "persistent": {
            "action.auto_create_index": "p_*",
            "cluster.max_shards_per_node":2147483647
        }
    }
    ```
    * 创建 index template
    ```
    PUT _template/p_
    {
        "index_patterns": [
            "p_*"
        ],
        "settings": {
            "refresh_interval" : "60s",
            "number_of_shards": 1,
            "number_of_replicas": 0
        },
        "mappings": {
            "_doc": {
                "_source": {
                    "enabled": true
                },
                "properties": {
                    "ts": {
                        "type": "date"
                    },
                    "id": {
                        "type": "keyword"
                    },
                    "name": {
                        "type": "keyword"
                    },
                    "type": {
                        "type": "keyword"
                    },
                    "output": {
                        "type": "keyword"
                    }
                }
            }
        }
    }
    ```

* 机器配置
    * cpu:Intel(R) Xeon(R) CPU E5-2620 0 @ 2.00GHz 24核
    * memory: 125g ，clock speed 未知 ，24 插槽
    * disk:HDD，型号未知，读取速度  460.92 MB/s，写入速度约 876 MB/s

* 使用 elasticsearch-rest-high-level-client：6.8.2
* ES 配置
    * jvm.options:
    ``` properties
    -Xms4g
    -Xmx4g
    ```
    * 其他为默认配置

* 查询脚本

    使用 Date Range Histogram + Top Hit 做 Downsampling，使用 term aggregator 做DownSampling笛卡尔积

    * 笛卡尔积示例
    ```
    {
        "size": 0,
        "query": {
            "bool": {
                "must": [
                    {
                        "range": {
                            "ts": {
                                "from": 1565751798176,
                                "to": 1565838198176,
                                "include_lower": true,
                                "include_upper": true,
                                "boost": 1
                            }
                        }
                    },
                    {
                        "term": {
                            "id": {
                                "value": "post",
                                "boost": 1
                            }
                        }
                    }
                ],
                "adjust_pure_negative": true,
                "boost": 1
            }
        },
        "aggs": {
            "cp": {
                "composite": {
                    "size": 5000,
                    "sources": [
                        {
                            "type": {
                                "terms": {
                                    "field": "type"
                                }
                            }
                        },
                        {
                            "name": {
                                "terms": {
                                    "field": "name"
                                }
                            }
                        },
                        {
                            "id": {
                                "terms": {
                                    "field": "id"
                                }
                            }
                        },
                        {
                            "output": {
                                "terms": {
                                    "field": "output"
                                }
                            }
                        },
                        {
                            "ts": {
                                "date_histogram": {
                                    "field": "ts",
                                    "interval": "hour"
                                }
                            }
                        }
                    ]
                },
                "aggs": {
                    "first": {
                        "top_hits": {
                            "_source": {
                                "includes": [
                                    "ts"
                                ]
                            },
                            "size": 1
                        }
                    }
                }
            }
        }
    }
    ```
    

    * 无笛卡尔积示例
    ```
    {
        "size": 0,
        "query": {
            "range": {
                "ts": {
                    "gte": 1565751798176,
                    "lte": 1565838198176
                }
            }
        },
        "aggs": {
            "paged": {
                "composite": {
                    "size": 5000,
                    "sources": {
                        "ts": {
                            "date_histogram": {
                                "field": "ts",
                                "interval": "hour"
                            }
                        }
                    }
                },
                "aggs": {
                    "first": {
                        "top_hits": {
                            "_source": {},
                            "size": 1
                        }
                    }
                }
            }
        }
    }
    ```

注: 使用 composite aggregator 之后需要使用after进行分页查询（由于after_key始终都会返回，因此可以先判断bucket的size是否小于composite的size，如果小于则到了最后一页，不需要再查询）

tsdb | batch put(10,000,000 records,5000 per batch) |query range(1d,1h)| query range(30d,1h)|query range with tag(1d,1h)|query range with tag(30d,1h)
---- | ---| ---| ---| ---| ---| ---| ---| ---| ---| ---| ---
elasticsearch(无笛卡尔积) |516071ms|4959ms|14509ms|4613ms|9217ms|
elasticsearch(有笛卡尔积) |NA|928700ms|单个请求超时未返回（tiemout 240s)|74790ms|4297954ms
