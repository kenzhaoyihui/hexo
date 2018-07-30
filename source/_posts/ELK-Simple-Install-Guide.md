title: ELK Simple Install Guide
author: Kenzhaoyihui
tags:
  - ELK
categories:
  - ELK
date: 2018-07-12 18:52:00
---
## ELK
##### Elasticsearch, Logstash, Kibana

1. Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

2. Logstash是一个完全开源的工具，他可以对你的日志进行收集、过滤，并将其存储供以后使用（如，搜索）。

3. Kibana 也是一个开源和免费的工具，它Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

Link to Download the ELK: https://www.elastic.co/downloads/



### 1. Elasticsearch

#####  `./bin/elasticsearch `

From the browser:  http://127.0.0.1:9200
```
{
  "name" : "VRTTWx8",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "j037X3RVSvyYz5Vq6PH5fA",
  "version" : {
    "number" : "6.3.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "eb782d0",
    "build_date" : "2018-06-29T21:59:26.107521Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}


```

Elasticsearch is OK!

Of course, you can install the `elasticsearch-head` to manage or check the elasticsearch information.

```
wget  https://github.com/mobz/elasticsearch-head/archive/master.zip
unzip master.zip
cd elasticsearch-head-master/
sudo npm install -g grunt-cli
sudo npm install

grunt server

```

###### elasticsearch_head_master
![head-master1](/images/elasticsearch_head2.png)


![head_master2](/images/elasticsearch_head1.png)


### 2. Logstash
Need to install log4j plugin : `./bin/logstash-plugin install logstash-input-log4j`
```
  log4j {
    mode => "server"
    host => "localhost"
    port => 4560
  }
}
filter {
  # Matched data are send to output.
}
output {
    elasticsearch {
    action => "index"         
    hosts  => "localhost:9200"   
    index  => "application_log" 
  }
}
```

#### 启动 `./bin/logstash -f config/log4j_to_es.conf `

From the browser: http://127.0.0.1:9600/
```
{"host":"hah","version":"6.3.1","http_address":"127.0.0.1:9600","id":"f912dc57-ace4-4efc-8435-b8a282909a84","name":"hah","build_date":"2018-06-29T22:43:59Z","build_sha":"b79493047db01afca1e11c856fe8538d7ecb5787","build_snapshot":false}
```

Logstash is OK!


### 3. Kibana

#### 启动 `./bin/kibana `

From the browser:  http://127.0.0.1:5601


![kibana](/images/kibana.png)