## Elasticsearch 单点安装

### 简介

Elasticsearch（简称ES）是一个分布式、可扩展、实时的搜索与数据分析引擎。ES不仅仅只是全文搜索，还支持结构化搜索、数据分析、复杂的语言处理、地理位置和对象间关联关系等。

ES的底层依赖Lucene，Lucene可以说是当下最先进、高性能、全功能的搜索引擎库。
但是Lucene仅仅只是一个库。为了充分发挥其功能，你需要使用Java并将Lucene直接集成到应用程序中。
更糟糕的是，您可能需要获得信息检索学位才能了解其工作原理，因为Lucene非常复杂。
鉴于Lucene如此强大却难以上手的特点，诞生了ES。

ES也是使用Java编写的，它的内部使用Lucene做索引与搜索，它的目的是隐藏Lucene的复杂性，取而代之的提供一套简单一致的RESTful API。

ES具有如下特点：

- 一个分布式的实时文档存储引擎，每个字段都可以被索引与搜索
- 一个分布式实时分析搜索引擎，支持各种查询和聚合操作
- 能胜任上百个服务节点的扩展，并可以支持PB级别的结构化或者非结构化数据

### 使用场景

关系型数据库有什么问题？
传统的关系数据库提供事务保证，具有不错的性能，高可靠性，久经历史考验，而且使用简单，功能强大，同时也积累了大量的成功案例。
后来，随着访问量的上升，几乎大部分使用 MySQL 架构的网站在数据库上都开始出现了性能问题，web 程序不再仅仅专注在功能上，同时也在追求性能。

读写分离
由于数据库的写入压力增加，读写集中在一个数据库上让数据库不堪重负，大部分网站开始使用主从复制技术来达到读写分离，以提高读写性能和读库的可扩展性。Mysql 的 master-slave 模式成为这个时候的网站标配了。

分表分库
开始流行使用分表分库来缓解写压力和数据增长的扩展问题。这个时候，分表分库成了一个热门技术，也是业界讨论的热门技术问题。

MySQL 的扩展性瓶颈
大数据量高并发环境下的 MySQL 应用开发越来越复杂，也越来越具有技术挑战性。分表分库的规则把握都是需要经验的。虽然有像淘宝这样技术实力强大的公司开发了透明的中间件层来屏蔽开发者的复杂性，但是避免不了整个架构的复杂性。分库分表的子库到一定阶段又面临扩展问题。还有就是需求的变更，可能又需要一种新的分库方式。

关系数据库很强大，但是它并不能很好的应付所有的应用场景。MySQL 的扩展性差（需要复杂的技术来实现），大数据下 IO 压力大，表结构更改困难，正是当前使用 MySQL 的开发人员面临的问题。

1. ElasticSearch有什么优势？
   非关系型、搜索引擎、近实时搜索与分析、高可用、天然分布式、横向可扩展
2. ES使用场景

**搜索引擎**
电商网站的商品搜索、站内搜索、模糊查询、全文检索服务

**非关系型数据库**
业务宽表（数据库字段太多，查询太慢，索引没有办法再做优化）
数据库做统计查询

**大数据近实时分析引擎**

**日志分析**

### 下载安装

```
## 下载解压
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.5.4.tar.gz
tar -xzf elasticsearch-6.5.4.tar.gz -C /usr/local
cd /usr/local/

## 配置环境变量 vi /etc/profile写入以下2行，保存退出后source /etc/profile
export ES_HOME=/usr/local/elasticsearch-6.5.4
export PATH=$ES_HOME/bin:$PATH

## 配置用户，es不能在root用户下启动
groupadd es
useradd -g es es
passwd es
chmod 777 elasticsearch-6.5.4
chown -R es:es elasticsearch-6.5.4

## es_home/config/elasticsearch.yml中增肌以下几行
xpack.ml.enabled: false
xpack.security.enabled: false
#xpack.security.enrollment.enabled: true
xpack.security.http.ssl.enabled: false

http.host: 0.0.0.0

## 启动

./elasticsearch 

## 后台启动
./elasticsearch -d

## 验证安装是否成功，用浏览器或者curl都可以
[root@localhost elasticsearch-6.5.4]# curl localhost:9200
{
  "name" : "BU9CO4N",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "3Lodrt8NQSeY4z1Du703gg",
  "version" : {
    "number" : "6.5.4",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "d2ef93d",
    "build_date" : "2018-12-17T21:17:40.758843Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
[root@localhost elasticsearch-6.5.4]# 
```

## CentOS下安装Kibana

### 前言

Kibana是一个开源的数据分析和可视化平台，通常与Elasticsearch一起使用，用于展示和分析大规模数据集。以下是关于Kibana的一些主要特点和功能：

数据可视化： Kibana允许用户将数据转化为交互式、实时的图形和可视化展示，包括折线图、柱状图、地图、仪表盘等。
数据查询与搜索： 用户可以在Kibana中进行数据查询和搜索，通过使用Elasticsearch的强大搜索引擎来查找和分析数据。
实时监控： Kibana提供实时监控功能，用户可以追踪系统的性能、日志、指标和其他关键信息。
仪表盘创建： 用户可以创建个性化的仪表盘，以展示重要的指标和数据，以帮助监控和分析业务绩效。
Elasticsearch集成： Kibana与Elasticsearch无缝集成，可以利用Elasticsearch的全文搜索、分布式存储和数据检索功能。
插件扩展性： Kibana支持插件，允许用户自定义和扩展平台以满足其独特的需求。
安全性： Kibana具有用户认证、访问控制和数据加密等安全功能，以保护数据和系统。
开源和社区支持： Kibana是开源项目，拥有一个活跃的社区，用户可以获取免费的支持、更新和插件。
Kibana通常与Elastic Stack的其他组件（如Elasticsearch、Logstash和Beats）一起使用，用于实时搜索、分析和可视化大规模的日志、指标和其他数据，以帮助组织进行决策、监控性能和发现问题。它在各种领域，包括日志管理、监控、安全信息和大数据分析中得到广泛应用。

### 下载

去官网下载安装包

**注意：kibana 的版本要与 ElasticSearch 的版本一致（不要ES用7.9.2，Kibana用6.8.3这类的操作），参考上一篇文章ElasticSearch 的安装**

### 拷贝

```shell
tar -zxvf kibana-7.9.2-linux-x86_64.tar.gz
```

### 修改配置

```shell
server.host: "0.0.0.0"........
elasticsearch.hosts: "http://localhost:9200" # 实际的es地址........
i18n.locale: "zh-CN"
```

### 启动 kibana

```shell
cd ./bin 
# --allow-root 表示允许以root用户运行 &是后台启动，Kibana没有 -d命令
./kibana --allow-root &
```

### 错误

启动时，可能会出现如下错误（没有错误的就不要看这块儿了）：

```text
[fatal][root] Error: Setup lifecycle of “monitoring” plugin wasn’t completed in 30sec. Consider disabling the plugin and re-start.
```

执行

```shell
vim ./config/kibana.yml
```

在最下面加上

```shell
monitoring.enabled: false
```







