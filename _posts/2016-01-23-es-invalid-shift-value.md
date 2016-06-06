---
layout: post
title: "ElasticSearch学习问题记录——Invalid shift value in prefixCoded bytes (is encoded value really an INT?)"
date: 2016-01-23
---


 &#160; &#160; &#160; &#160; 最近在做一个电商项目，其中商品搜索中出现一个奇怪的现象，根据某个字段排序的时候会出现商品数量减少的情况。按照一般路要么查不出来，要么正常显示，为什么增加了按照销量排序就会出现查询结果减少的情况。   
  &#160; &#160; &#160; &#160; 查了下ES日志发现有报错：nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)
  &#160; &#160; &#160; &#160; 看得出是数据类型转换错误，但是为什么错还是不清楚。求助GOOGLE。发现下面一则和我的情形相同。   
https://github.com/elastic/elasticsearch/issues/9191#issuecomment-77784022

 &#160; &#160; &#160; &#160; 查看了下索引中索引类型映射到数据结构，发现确实存在同名字段不同类型的情况。将错误的索引类型删除，保持同一结构，问题就解决了。   

 &#160; &#160; &#160; &#160; 为了进一步证实，我又写了一个小例子。先制造些错误，school使用先设置类型映射在添加数据。school2直接加数据，类型自动识别。下面是索引类型的映射结构:   
{% highlight json %}   
{
    "state": "open",
    "settings": {
        "index.version.created": "901399",
        "index.number_of_replicas": "1",
        "index.uuid": "qK4PV5IsQZm8eb1QCU7b6w",
        "index.number_of_shards": "5"
    },
    "mappings": {
        "school2": {
            "properties": {
                "id": {
                    "type": "string"
                },
                "name": {
                    "type": "string"
                },
                "age": {
                    "type": "long"
                },
                "studentList": {
                    "properties": {
                        "sex": {
                            "type": "string"
                        },
                        "studentId": {
                            "type": "string"
                        },
                        "studentName": {
                            "type": "string"
                        }
                    }
                }
            }
        },
        "school": {
            "properties": {
                "id": {
                    "store": true,
                    "analyzer": "ik",
                    "type": "string"
                },
                "name": {
                    "store": true,
                    "analyzer": "ik",
                    "type": "string"
                },
                "age": {
                    "store": true,
                    "type": "integer"
                },
                "studentList": {
                    "properties": {
                        "sex": {
                            "store": true,
                            "analyzer": "ik",
                            "type": "string"
                        },
                        "studentId": {
                            "store": true,
                            "analyzer": "ik",
                            "type": "string"
                        },
                        "studentName": {
                            "store": true,
                            "analyzer": "ik",
                            "type": "string"
                        }
                    },
                    "type": "nested"
                }
            }
        }
    },
    "aliases": []
}   
{% endhighlight %}   

注意索引类型school和school2中age字段的类型前者是Integer后者是long。我们按照全匹配查询并根据age进行排序。问题产生了。下面就是报错的结果。  
{% highlight json %} 
{
    "error": "SearchPhaseExecutionException[Failed to execute phase [query], all shards failed; shardFailures {[iJhim0-hSMKr_uHZFrBfTA][demoindex][3]: RemoteTransportException[[node2][inet[/192.168.0.202:9300]][search/phase/query]]; nested: QueryPhaseExecutionException[[demoindex][3]: query[filtered(ConstantScore(*:*))->cache(_type:school2)],from[0],size[20],sort[<custom:\"age\": org.elasticsearch.index.fielddata.fieldcomparator.LongValuesComparatorSource@32780f73>!]: Query Failed [Failed to execute main query]]; nested: ElasticSearchException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: UncheckedExecutionException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; }{[0P8agoQGSS-h4abY0hMI8Q][demoindex][4]: QueryPhaseExecutionException[[demoindex][4]: query[filtered(ConstantScore(*:*))->cache(_type:school2)],from[0],size[20],sort[<custom:\"age\": org.elasticsearch.index.fielddata.fieldcomparator.LongValuesComparatorSource@1d42c789>!]: Query Failed [Failed to execute main query]]; nested: ElasticSearchException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: UncheckedExecutionException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; }{[0P8agoQGSS-h4abY0hMI8Q][demoindex][1]: QueryPhaseExecutionException[[demoindex][1]: query[filtered(ConstantScore(*:*))->cache(_type:school2)],from[0],size[20],sort[<custom:\"age\": org.elasticsearch.index.fielddata.fieldcomparator.LongValuesComparatorSource@2998a407>!]: Query Failed [Failed to execute main query]]; nested: ElasticSearchException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: UncheckedExecutionException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; }{[iJhim0-hSMKr_uHZFrBfTA][demoindex][2]: RemoteTransportException[[node2][inet[/192.168.0.202:9300]][search/phase/query]]; nested: QueryPhaseExecutionException[[demoindex][2]: query[filtered(ConstantScore(*:*))->cache(_type:school2)],from[0],size[20],sort[<custom:\"age\": org.elasticsearch.index.fielddata.fieldcomparator.LongValuesComparatorSource@65fde78f>!]: Query Failed [Failed to execute main query]]; nested: ElasticSearchException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: UncheckedExecutionException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; }{[iJhim0-hSMKr_uHZFrBfTA][demoindex][0]: RemoteTransportException[[node2][inet[/192.168.0.202:9300]][search/phase/query]]; nested: QueryPhaseExecutionException[[demoindex][0]: query[filtered(ConstantScore(*:*))->cache(_type:school2)],from[0],size[20],sort[<custom:\"age\": org.elasticsearch.index.fielddata.fieldcomparator.LongValuesComparatorSource@f6e47a1>!]: Query Failed [Failed to execute main query]]; nested: ElasticSearchException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: UncheckedExecutionException[java.lang.NumberFormatException: Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; nested: NumberFormatException[Invalid shift value in prefixCoded bytes (is encoded value really an INT?)]; }]",
    "status": 500
}  
{% endhighlight %}    
