---
layout: post # 使用post模版
title:  "ElasticSearch"
date:   2021-03-22
categories: es
---

# ElasticSearch

## 什么是ElasticSearch

Elasticsearch是一个实时的分布式搜索和分析引擎。它可以帮助你用前所未有的速度去处理大规模数据。ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

### 什么是索引

> 是否索引，就是看该域是否能被搜索。

> 是否分词，就表示搜索的时候是整体匹配还是单词匹配

> 是否存储，就是是否在页面上显示

## ElasticSearch特点

（1）可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司；也可以运行在单机上

（2）将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES；

（3）开箱即用的，部署简单

（4）全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理

## ElasticSearch体系结构

下表是Elasticsearch与MySQL数据库逻辑结构概念的对比

| Elasticsearch  | 关系型数据库Mysql |
| -------------- | ----------------- |
| 索引(index)    | 数据库(databases) |
| 类型(type)     | 表(table)         |
| 文档(document) | 行(row)           |

## ElasticSearch部署与启动

无需安装，解压安装包后即可使用

在命令提示符下，进入ElasticSearch安装目录下的bin目录,执行命令

```cmd
elasticsearch
```

即可启动。

我们打开浏览器，在地址栏输入http://127.0.0.1:9200/ 即可看到输出结果

```json
{
    "name": "VnIzUEv",
    "cluster_name": "elasticsearch",
    "cluster_uuid": "eR1v7tfFRBqAe3UgHN7xkA",
    "version": {
        "number": "5.6.8",
        "build_hash": "688ecce",
        "build_date": "2018-02-16T16:46:30.010Z",
        "build_snapshot": false,
        "lucene_version": "6.6.1"
    },
    "tagline": "You Know, for Search"
}
```

## Postman调用RestAPI

### 新建索引

例如我们要创建一个叫**articleindex**的索引 ,就以**put**方式提交

```http
http://127.0.0.1:9200/articleindex/
```

```json
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "articleindex"
}
```

### 新建文档

新建文档：

以post方式提交 http://127.0.0.1:9200/articleindex/article

```http
http://127.0.0.1:9200/articleindex/article
```

body:

```json
{
	"title":"SpringBoot2.0",
	"content":"发布啦"	
}
```

返回结果如下：

```json
{
    "_index": "articleindex",
    "_type": "article",
    "_id": "AXhP2zDIRJlPXzAjbo4U",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "created": true
}
```


_id是由系统自动生成的。 为了方便之后的演示，我们再次录入几条测试数据。

### 查询全部文档

查询某索引某类型的全部数据，以**get**方式请求

```url
http://127.0.0.1:9200/articleindex/article/_search
```

 返回结果如下：

```json
{
    "took": 36,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1.0,
        "hits": [
            {
                "_index": "articleindex",
                "_type": "article",
                "_id": "AXhP2zDIRJlPXzAjbo4U",
                "_score": 1.0,
                "_source": {
                    "title": "SpringBoot2.0",
                    "content": "发布啦"
                }
            }
        ]
    }
}
```

### 修改文档

以**put**形式提交以下地址：

```http
http://127.0.0.1:9200/articleindex/article/AXhP2zDIRJlPXzAjbo4U
```

body:

```json
{
	"title":"SpringBoot2.0正式版",
	"content":"发布了吗"
}
```

返回结果：

```json
{
    "_index": "articleindex",
    "_type": "article",
    "_id": "AXhP2zDIRJlPXzAjbo4U",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "created": false
}
```

再次全部查询

```json
{
    "took": 2,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1.0,
        "hits": [
            {
                "_index": "articleindex",
                "_type": "article",
                "_id": "AXhP2zDIRJlPXzAjbo4U",
                "_score": 1.0,
                "_source": {
                    "title": "SpringBoot2.0正式版",
                    "content": "发布了吗"
                }
            }
        ]
    }
}
```

如果我们在地址中的ID不存在，则会创建新文档

以**put**形式提交以下地址

```http
http://127.0.0.1:9200/articleindex/article/1
```

body:

```json
{
	"title":"十次方课程好给力",
	"content":"知识点很多"
}
```

返回信息：

```json
{
    "_index": "articleindex",
    "_type": "article",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "created": true
}
```

### 按ID查询文档

GET方式请求

```json
http://127.0.0.1:9200/articleindex/article/1
```

### 基本匹配查询

根据某列进行查询 get方式提交下列地址：

```json
http://127.0.0.1:9200/articleindex/article/_search?q=title:十次方课程好给力
```

以上为按标题查询，返回结果如下：

```json
{
    "took": 13,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 2.0649285,
        "hits": [
            {
                "_index": "articleindex",
                "_type": "article",
                "_id": "1",
                "_score": 2.0649285,
                "_source": {
                    "title": "十次方课程好给力",
                    "content": "知识点很多"
                }
            }
        ]
    }
}
```



### 模糊查询

我们可以用*代表任意字符：

```http
http://127.0.0.1:9200/articleindex/article/_search?q=title:*s*
```

返回值

```json
{
    "took": 14,
    "timed_out": false,
    "_shards": {
        "total": 5,
        "successful": 5,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": 1,
        "max_score": 1.0,
        "hits": [
            {
                "_index": "articleindex",
                "_type": "article",
                "_id": "AXhP2zDIRJlPXzAjbo4U",
                "_score": 1.0,
                "_source": {
                    "title": "SpringBoot2.0正式版",
                    "content": "发布了吗?发布你妈"
                }
            }
        ]
    }
}
```

### 删除文档

根据ID删除文档,删除ID为1的文档 DELETE方式提交

```http
http://192.168.184.134:9200/articleindex/article/1
```

返回结果如下：

```json
{
    "found": true,
    "_index": "articleindex",
    "_type": "article",
    "_id": "1",
    "_version": 2,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    }
}
```



## Head插件安装与使用

### Head插件安装

> 如果都是通过rest请求的方式使用Elasticsearch，未免太过麻烦，而且也不够人性化。我
> 们一般都会使用图形化界面来实现Elasticsearch的日常管理，最常用的就是Head插件

步骤1：

下载head插件：https://github.com/mobz/elasticsearch-head

配套资料中已提供。 elasticsearch-head-master.zip

步骤2：

解压到任意目录，但是要和elasticsearch的安装目录区别开。

步骤3：

安装node js ,安装cnpm

```cmd
npm install ‐g cnpm ‐‐registry=https://registry.npm.taobao.org
```

步骤4：

将grunt安装为全局命令 。Grunt是基于Node.js的项目构建工具。它可以自动运行你所设定的任务

```cmd
npm install ‐g grunt‐cli
```

步骤5：安装依赖

```cmd
cnpm install
```

步骤6：

进入head目录启动head，在命令提示符下输入命令

```cmd
grunt server
```

步骤7：

打开浏览器，输入 http://localhost:9100

步骤8：

点击连接按钮没有任何相应，按F12发现有如下错误

```http
No 'Access-Control-Allow-Origin' header is present on the requested resource
```

这个错误是由于elasticsearch默认不允许跨域调用，而elasticsearch-head是属于前端工程，所以报错。我们这时需要修改elasticsearch的配置，让其允许跨域访问。修改elasticsearch配置文件：elasticsearch.yml，增加以下两句命令：

```yaml
http.cors.enabled: true
http.cors.allow‐origin: "*"
```

此步为允许elasticsearch跨越访问 点击连接即可看到相关信息

## Head插件操作

## IK分词器

### 什么是IK分词器

我们在浏览器地址栏输入浏览器显示效果如下

```http
http://127.0.0.1:9200/_analyze?analyzer=chinese&pretty=true&text=我是程序员，
```

```json

{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "程",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "序",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "员",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    }
  ]
}
```

默认的中文分词是将每个字看成一个词，这显然是不符合要求的，所以我们需要安装中文分词器来解决这个问题。IK分词是一款国人开发的相对简单的中文分词器。虽然开发者自2012年之后就不在维护了，但在工程应用中IK算是比较流行的一款！我们今天就介绍一下IK中文分词器的使用。

### IK分词器安装(安装路径不要有空格)

下载地址：https://github.com/medcl/elasticsearch-analysis-ik/releases 下载5.6.8版本课程配套资源也提供了: 资源\配套软件\elasticsearch\elasticsearch-analysis-ik-5.6.8.zip

（1）先将其解压，将解压后的elasticsearch文件夹重命名文件夹为ik

（2）将ik文件夹拷贝到elasticsearch/plugins 目录下。

（3）重新启动，即可加载IK分词器

### IK分词器测试

IK提供了两个分词算法ik_smart 和 ik_max_word

其中 ik_smart 为最少切分，ik_max_word为最细粒度划分

我们分别来试一下

（1）最小切分：在浏览器地址栏输入地址

http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=我是程序员

输出的结果为：

```json

{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "程序员",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}
```

（2）最细切分：在浏览器地址栏输入地址

```http
http://127.0.0.1:9200/_analyze?analyzer=ik_max_word&pretty=true&text=我是程序员
```

```json

{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "是",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "程序员",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "程序",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "员",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "CN_CHAR",
      "position" : 4
    }
  ]
}
```

### 自定义词库

我们现在测试"传智播客"，浏览器的测试效果如下：

```http
http://127.0.0.1:9200/_analyze?analyzer=ik_smart&pretty=true&text=传智播客
```

默认的分词并没有识别“传智播客”是一个词。如果我们想让系统识别“传智播客”是一个词，需要编辑自定义词库。

步骤：

（1）进入elasticsearch/plugins/ik/config目录

（2）新建一个my.dic文件，编辑内容：

```cmd
传智播客
```

修改**IKAnalyzer.cfg.xml**（在**ik/config**目录下）

```properties
<properties>
	<comment>IK Analyzer 扩展配置</comment>
	<!‐‐用户可以在这里配置自己的扩展字典 ‐‐>
	<entry key="ext_dict">my.dic</entry>
	<!‐‐用户可以在这里配置自己的扩展停止词字典‐‐>
	<entry key="ext_stopwords"></entry>
</properties>
```

再次测试

```json
{
  "tokens" : [
    {
      "token" : "传智播客",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 0
    }
  ]
}
```

## SpringBoot 引入

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-elasticsearch</artifactId>
</dependency>
```

## elasticsearch与MySQL数据同步

### Logstash

#### 什么是Logstash

Logstash是一款轻量级的日志搜集处理框架，可以方便的把分散的、多样化的日志搜集起来，并进行自定义的处理，然后传输到指定的位置，比如某个服务器或者

文件。

Logstash安装与测试：

解压，进入bin目录

```cmd
logstash ‐e 'input { stdin { } } output { stdout {} }'
```

控制台输入字符，随后就有日志输出

```cmd
PS D:\Program\elasticsearch-5.6.8\logstash-5.6.8\bin> .\logstash -e 'input { stdin { } } output { stdout {} }'
Sending Logstash's logs to D:/Program/elasticsearch-5.6.8/logstash-5.6.8/logs which is now configured via log4j2.properties
[2021-03-21T17:10:45,474][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/modules/fb_apache/configuration"}
[2021-03-21T17:10:45,477][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/modules/netflow/configuration"}
[2021-03-21T17:10:45,485][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.queue", :path=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/data/queue"}
[2021-03-21T17:10:45,486][INFO ][logstash.setting.writabledirectory] Creating directory {:setting=>"path.dead_letter_queue", :path=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/data/dead_letter_queue"}
[2021-03-21T17:10:45,496][INFO ][logstash.agent           ] No persistent UUID file found. Generating new UUID {:uuid=>"01b383e5-5220-4d5a-af0b-af188025f892", :path=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/data/uuid"}
[2021-03-21T17:10:45,606][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>16, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>2000}
[2021-03-21T17:10:45,630][INFO ][logstash.pipeline        ] Pipeline main started
The stdin plugin is now waiting for input:
[2021-03-21T17:10:45,696][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
```

stdin，表示输入流，指从键盘输入

stdout，表示输出流，指从显示器输出

命令行参数:

-e 执行

--config 或 -f 配置文件，后跟参数类型可以是一个字符串的配置或全路径文件名或全路径路径(如：/etc/logstash.d/，logstash会自动读取/etc/logstash.d/目录下所有*.conf 的文本文件，然后在自己内存里拼接成一个完整的大配置文件再去执行)

### MySQL数据导入Elasticsearch

（1）在logstash-5.6.8安装目录下创建文件夹mysqletc （名称随意）

（2）文件夹下创建mysql.conf （名称随意） ，内容如下：

```properties
input {
	jdbc {
		# mysql jdbc connection string to our backup databse 后面的test对应mysql中的test数据库
		jdbc_connection_string => "jdbc:mysql://192.168.52.128:3306/tensquare_article?characterEncoding=UTF8"
		# the user we wish to excute our statement as
		jdbc_user => "root"
		jdbc_password => "123456"
		# the path to our downloaded jdbc driver
		jdbc_driver_library => "D:/Program/elasticsearch-5.6.8/logstash-5.6.8/mysqletc/mysql-connector-java-5.1.46.jar"
		# the name of the driver class for mysql
		jdbc_driver_class => "com.mysql.jdbc.Driver"
		jdbc_paging_enabled => "true"
		jdbc_page_size => "50000"
		#以下对应着要执行的sql的绝对路径。
		statement => "select id,title,content from tb_article"
		#定时字段 各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
		schedule => "* * * * *"
	}
}
output {
	elasticsearch {
		#ESIP地址与端口
		hosts => "localhost:9200"
		#ES索引名称（自己定义的）
		index => "tensquare"
		#自增ID编号
		document_id => "%{id}"
		document_type => "article"
	}
	stdout {
		#以JSON格式输出
		codec => json_lines
	}
}

```

（3）将mysql驱动包mysql-connector-java-5.1.46.jar拷贝至D:/logstash-5.6.8/mysqletc/ 下 。D:/logstash-5.6.8是你的安装目录

（4）命令行下执行

```cmd
logstash ‐f ../mysqletc/mysql.conf
```

观察控制台输出，每间隔1分钟就执行一次sql查询。

```cmd
PS D:\Program\elasticsearch-5.6.8\logstash-5.6.8\bin> .\logstash -f ..\mysqletc\mysql.conf
Sending Logstash's logs to D:/Program/elasticsearch-5.6.8/logstash-5.6.8/logs which is now configured via log4j2.properties
[2021-03-21T17:20:37,341][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"fb_apache", :directory=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/modules/fb_apache/configuration"}
[2021-03-21T17:20:37,343][INFO ][logstash.modules.scaffold] Initializing module {:module_name=>"netflow", :directory=>"D:/Program/elasticsearch-5.6.8/logstash-5.6.8/modules/netflow/configuration"}
[2021-03-21T17:20:37,744][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://localhost:9200/]}}
[2021-03-21T17:20:37,745][INFO ][logstash.outputs.elasticsearch] Running health check to see if an Elasticsearch connection is working {:healthcheck_url=>http://localhost:9200/, :path=>"/"}
[2021-03-21T17:20:37,794][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://localhost:9200/"}
[2021-03-21T17:20:37,817][INFO ][logstash.outputs.elasticsearch] Using mapping template from {:path=>nil}
[2021-03-21T17:20:37,820][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>50001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"_all"=>{"enabled"=>true, "norms"=>false}, "dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date", "include_in_all"=>false}, "@version"=>{"type"=>"keyword", "include_in_all"=>false}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2021-03-21T17:20:37,824][INFO ][logstash.outputs.elasticsearch] Installing elasticsearch template to _template/logstash
[2021-03-21T17:20:37,859][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["//localhost:9200"]}
[2021-03-21T17:20:37,860][INFO ][logstash.pipeline        ] Starting pipeline {"id"=>"main", "pipeline.workers"=>16, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>5, "pipeline.max_inflight"=>2000}
[2021-03-21T17:20:38,025][INFO ][logstash.pipeline        ] Pipeline main started
[2021-03-21T17:20:38,088][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
[2021-03-21T17:21:00,486][INFO ][logstash.inputs.jdbc     ] (0.005000s) SELECT version()
[2021-03-21T17:21:00,495][INFO ][logstash.inputs.jdbc     ] (0.005000s) SELECT count(*) AS `count` FROM (select id,title,content from tb_article) AS `t1` LIMIT 1
[2021-03-21T17:21:00,498][INFO ][logstash.inputs.jdbc     ] (0.002000s) SELECT * FROM (select id,title,content from tb_article) AS `t1` LIMIT 50000 OFFSET 0
{"@version":"1","id":"1373110975301029888","@timestamp":"2021-03-21T09:21:00.507Z","title":"string","content":"string"}
```

## Docker 部署Elasticsearch

```cmd
docker pull elasticsearch:5.6.8
docker run ‐di ‐‐name=tensquare_elasticsearch ‐p 9200:9200 ‐p 9300:9300 elasticsearch:5.6.8
```

```http
http://192.168.52.128:9200/
http://192.168.52.128:9300/
```

配置文件映射

```cmd
docker exec ‐it tensquare_elasticsearch /bin/bash
```

拷贝配置文件到宿主机

```cmd
# 拷贝配置文件到宿主机
docker cp tensquare_elasticsearch:/usr/share/elasticsearch/config/elasticsearch.yml /usr/share/elasticsearch.yml
# 删除
docker stop tensquare_elasticsearch
docker rm tensquare_elasticsearch
# 创建镜像并 映射文件目录
docker run -di --name=tensquare_elasticsearch --restart=always -p 9200:9200 -p 9300:9300 -v /usr/share/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml elasticsearch:5.6.8
```

修改/usr/share/elasticsearch.yml

```yaml
transport.host: 0.0.0.0
```

系统调优

修改/etc/security/limits.conf ，追加内容

```properties
* soft nofile 65536
* hard nofile 65536
```

nofile是单个进程允许打开的最大文件个数 soft nofile 是软限制 hard nofile是硬限制修改/etc/sysctl.conf，追加内容

```properties
vm.max_map_count=655360
```

限制一个进程可以拥有的VMA(虚拟内存区域)的数量 执行下面命令 修改内核参数马上生效

```properties
sysctl ‐p
```

### IK分词器安装

```cmd
# 拷贝文件至docker宿主机
PS D:\Program\elasticsearch-5.6.8\plugins> scp -r ik/ root@192.168.52.128:/root/upload
# 将文件拷贝到docker容器
docker cp ik tensquare_elasticsearch:/usr/share/elasticsearch/plugins/
# 重启生效
docker restart tensquare_elasticsearch
```

### HEAD插件安装

（1）修改/usr/share/elasticsearch.yml ,添加允许跨域配置

```yaml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

```cmd
# 下载镜像
docker pull mobz/elasticsearch-head:5
# 运行
docker run -di --name=myhead --restart=always -p 9100:9100 mobz/elasticsearch-head:5 
```

### logstash

```cmd
# 拉取
docker pull logstash:7.5.1
# 创建
docker run -itd \
-v /opt/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf \
-v /opt/logstash/mysql-connector-java-5.1.46.jar:/app/mysql-connector-java-5.1.46.jar \
--name=test_logstash \
logstash:5.6.8
```