# ElasticSearch
***

## 1. 基础

### 1.1 ElasticSearch简介

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElasticSearch（简称ES），是一个分布式、可扩展、准实时的搜索与数据分析引擎，除了强大的全文索引，ES还支持结构化索引、数据分析、海量数据的准实处理等功能。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Lucene是一个高性能、全功能的搜索引擎库。ES内部使用Lucene实现索引和搜索，目的是隐藏Lucene的复杂性，对外提供简单统一的RESTful API。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElasticSearch优点如下：
1. 功能丰富：强大的全文索引、海量数据准实时处理、同义词处理、相关度排名、复杂数据分析等
2. 高可拓展性：ES可以运行在单机上，也可以作为大型分布式集群运行，并且新添加服务器进入集群十分容易。
3. 高可用：复制分片的机制，保证了一些服务器宕机后，集群仍能正常工作。
4. 开箱即用：提供简单易用的 API，服务的搭建、部署和使用都很容易操作。

### 1.2 搭建学习环境
#### 1.2.1 安装ElasticSearch

1. 从 `www.elastic.co`官网下载并解压缩ElasticSearch 
2. 运行 `bin/elasticsearch` (Windows下运行`bin\elasticsearch.bat`) 
3. 运行 `curl http://localhost:9200/` ，如果成功返回ES相关信息即安装成功

#### 1.2.2 安装Kibana

演示用例均在Kibana中的开发工具-Console中运行，推荐安装
1. 从 `www.elastic.co`官网下载并解压缩Kibana
2.  修改config/kibana.yml
> 将elasticsearch.hosts修改为你的ES的地址（默认http://localhost:9200）
> 将i18n.locale设置为"zh-CN"中文（这个随意）
3. 运行 `bin/kibana` (Windows下运行 `bin\kibana.bat` ) 
4. 浏览器打开http://localhost:5601，能正常打开即安装成功

以下篇幅，我们将着重从分布式架构、全文索引、准实时搜索三个方面对ElasticSearch进行探索。

## 2. 分布式架构
### 2.1 节点（node）
一个节点就是一个ES实例，一个集群由一个或多个节点构成，他们拥有相同的`cluster.name`，他们协同工作，分享数据和负载，当加入或删除一个节点时，其它节点会自动感知并适应。
#### 2.1.1 节点类型
- 主节点（master node）
```
# conf/elasticsearch.yml
node.master:true
```
> 设置master为true后，其表示这个node是一个master的候选节点，可以参与选举。
> 集群中的一个节点会被选举为`master节点`，它将临时管理`集群级别`的变更（如新增、删除节点或新增、删除索引）。任何节点都可以成为主节点。

- 数据节点（data node）
```
# conf/elasticsearch.yml
node.master:false
node.data:true
```
> 数据节点是储存索引数据的节点，主要对文档进行增删改查、聚合等操作。数据节点对CPU、内存、io要求较高。

- 协调节点（client node）
```
# conf/elasticsearch.yml
node.master:false
node.data:false
```
> 当主节点和数据节点都设置为false时，该节点只能处理路由请求、处理搜索、分发索引等操作，本质上来说，这种节点像一个智能负载平衡器

- 提取节点（Ingest node）
```
# conf/elasticsearch.yml
node.ingest:true
```
> 对输入的数据进行处理和转换，如解析输入中的日期，或将输入中的字段值类型转化为指定类型。

- 部落节点（tribe node）
```
# conf/elasticsearch.yml
tribe:
	t1: 
		cluster.name: cluster_one
	t2: 
		cluster.name: cluster_two
``` 
> 部落节点可以跨越多个集群，它可以接收每个集群的状态，然后合并成一个全局集群的状态，它可以读写所有节点上的数据

**建议**：
1. 一个节点的默认配置是：主节点+数据节点两属性为一身，当集群中节点增多时，我们推荐将主节点和数据节点分离，这样做的好处是，当流量增长时，master节点不会成为整个系统的瓶颈，它需要处理的事情没有增加太多。
2. 独立的协调节点在一个较大的集群中是非常有用的，它中消除了数据节点的聚合/查询的请求解析和最终阶段，随着集群写入以及查询负载的增大，可以通过协调节点减轻数据节点的压力，可以让数据节点更多专注于数据的写入以及查询。
3. 对于Ingest节点，如果我们没有格式转换、类型转换等需求，直接设置为false。

#### 2.1.2 master节点选举策略
一个集群中可以有多个候选主节点，此时需要进行master选举，保证只有一个节点当选master。如果多个节点当选master，则集群会出现脑裂，脑裂会破坏数据一致性，导致集群不可控，进而产生非预期情况。
为了避免脑裂，ES才用了分布式系统常见的解决方案：选举出的master必须得到多数派的候选主节点认可，以此来保证选举只产生一个master。
集群多数派（quorum）通过以下配置来进行配置：
```
# conf/elasticsearch.yml
discovery.zen.minimum_master_nodes: (候选主节点数)/2+1
```

**master选举是谁发起的？什么时候发起？**

master选举是由候选主节点发起，当一个候选主节点发现满足以下条件时，发起选举：
1. 当前候选主节点不是master
2. 当前候选主节点通过ZenDiscovery模块的ping操作询问集群内其它已知节点时，无法发现master
3. 包括本节点在内，当前已有超过minimum_master_nodes个候选主节点没有连接到master。

**当发起选举时，选举谁？**

直接看源码，被选举的是排序后的第一个候选主节点。
```java
public MasterCandidate selectMaster(Collection<MasterCandidate> candidates) {
        assert hasEnoughCandidates(candidates);
        List<MasterCandidate> sortedCandidates = new ArrayList<>(candidates);
        sortedCandidates.sort(MasterCandidate::compare);
        return sortedCandidates.get(0);
}
```
compare源码，
```java
public static int compare(MasterCandidate c1, MasterCandidate c2) {
    // we explicitly swap c1 and c2 here. the code expects "better" is lower in a sorted
    // list, so if c2 has a higher cluster state version, it needs to come first.
    int ret = Long.compare(c2.clusterStateVersion, c1.clusterStateVersion);
    if (ret == 0) {
        ret = compareNodes(c1.getNode(), c2.getNode());
    }
    return ret;
}
```
如上面源码所示，先根据节点的clusterStateVersion比较，clusterStateVersion越大，优先级越高。clusterStateVersion相同时，进入compareNodes，其内部按照节点的Id比较(Id为节点第一次启动时随机生成)。

### 2.2 分片（shards）
#### 2.2.1 什么是分片
每一个分片都是一个`Lucene实例`，索引只是用来指向一个或多个分片的逻辑命名空间。文档被存储在分片中，文档在分片中被检索，但是应用不与分片直接通讯，而是与索引直接通讯。
可以把分片想象成数据的容器，文档存储在分片中，分片分配到集群中的节点上，当集群扩容或缩减时，ES会自动在节点间`迁移`分片，以保证集群平衡。
#### 2.2.2 分片类型
分片可以是`主分片`或`复制分片`，索引中的每一个文档属于一个单独的主分片，所以主分片的数量决定了索引最多能储存多少数据。
复制分片是主分片的副本，作用有两方面：
1. 防止硬件故障导致数据丢失
2. 可以提供读请求，或从其它分片取回数据

当索引建立完成时，主分片的数量就固定了，但是复制分片的数量可以随时调整。
#### 2.2.3 文档在分片上的路由
当你索引一个文档时，它被存储在单独的一个主分片上，ElasticSearch是如何知道文档属于哪个分片呢？
> 文档在分片上的路由由一个简单的算法决定：
> ```
> shard = hash(routing) % number_of_primary_shards
> ```
> routing是任意一个字符串，它默认为_id，也可以是自定义值。routing通过hash函数生成一个数字，然后除以主分片数量得到一个余数，余数的范围永远在[0, number_or_primary_shards - 1]内，这个余数就是该文档所在的分片。

这也就能解释为什么主分片数量只能在索引建立时指定且不能修改了
> 如果主分片的数量在未来发生了改变，所有先前的路由值都失效了，文档也就永远找不到了。
### 2.3  集群（cluster）
#### 2.3.1 集群健康
集群健康有三种级别：`green`、`yellow`、`red`
- green：所有主分片和复制分片都可用
- yellow：所有主分片都可用，但是不是所有复制分片都可用
- red：不是所有的主分片都可用

下面的命令可以查询 集群健康：
```java
//curl方式写法
curl -XGET "http://127.0.0.1:9200/_cluster/health"
//在console中这样简写
GET /_cluster/health
```
#### 2.3.2 集群内部结构
接下来我们将从一个单节点集群开始，逐步完善成一个高可用的ES集群。通过这个过程，理解分片、节点和集群的关系以及集群的负载均衡。

**1. 单节点集群**

创建一个名为test的索引，我们分配三个主分片和一个复制分片
```
PUT /test
{
	"settings":{
		"number_of_shards":3,
		"number_of_replicas":1
	}
}
```
此时你的集群长这个样子↓
![单节点3个分片的集群](https://raw.githubusercontent.com/201502lisihao/elasticSearch-/master/images/1.png "单节点3个分片的集群")

你的单节点集群的主节点将拥有三个主分片，此时集群已经可以正常处理请求了，集群的健康程度为yellow（不是所有的复制分片都可用）。目前集群是没有容灾能力的，主节点挂掉，数据将会丢失。

**2. 增加灾备能力**

此时，我们在另外一台机器上新增加一个节点（启动一个ES实例），并配置`conf/elasticsearch.yml`,使两个ES实例的master.name保持一致，并且保证两台机器可以互相通讯，ES会自动将第二个节点加入集群。

此时你的集群长这个样子↓
![2个节点的集群](https://raw.githubusercontent.com/201502lisihao/elasticSearch-/master/images/2.png "2个节点的集群")

我们观察到，node2加入了集群，复制分片也被分配在node2上。此时我们的文档在node1和node2上都可以被检索，这意味丢失任何一个node时，我们都可以保证数据的完整性和服务的可用性。

增加节点后再查看集群的health，为green，代表所有的主分片和复制分片都可用

**3. 横向拓展**

可拓展的系统一般分为两种，纵向拓展（更好的服务器配置）和横向拓展（更多的机器）。纵向拓展有很大局限性，真正的拓展应该是横向的，通过增加节点来均衡负载和增加可靠性。

此时，我们只需要在一台新机器上启动第三个节点（启动一个ES实例），配置master.name，我们的集群会自动感知到这个节点，将它加入集群，并重新分配负载。

此时你的集群长这个样子↓
![3个节点的集群](https://raw.githubusercontent.com/201502lisihao/elasticSearch-/master/images/3.png "3个节点的集群")

任意一节点故障后，剩余节点仍保留有全部的数据。

**4. 更多拓展**

主分片数量在创建索引时已经给定且无法更改，但是我们可以通过增加复制分片的方式，来拓展我们集群的规模。我们将复制分片数设置为2。

```
PUT /test/_settings
{
	"number_of_replicas":2
}
```

此时你的集群长这个样子↓
![3个主分片3个复制分片的3节点集群](https://raw.githubusercontent.com/201502lisihao/elasticSearch-/master/images/4.png "3个主分片3个复制分片的3节点集群")

我们观察到，test索引现在拥有9个分片（3个主分片，6个复制分片），这样我们的集群最多可以拓展到9个节点（每个节点一个分片）以拓展ES集群的搜索能力（每个分片获得一个节点的全部硬件资源，集群也获得了更大的吞吐能力）。

至此，一个高性能、高可用的ES集群就完成啦。

## 3. 全文检索
### 3.1 什么是全文检索
全文检索是一种将文件中所有文本与检索项匹配的文字资料检索方法。全文检索首先将要查询的目标文档中的词提取出来，组成索引，通过查询索引达到搜索目标文档的目的。

这种先建立索引，再对索引进行搜索的过程就叫全文检索（Full-text Search）。

### 3.2 如何做到全文检索
Elasticsearch的主要强项还是在全文检索方面，Github上的数据搜索就是使用ElasticSearch实现的，ES帮助Github在全站海量项目代码中实现关键词搜索，响应速度也是十分喜人。

ElasticSearch使用一种称为`倒排索引`的结构，它适用于快速的全文搜索。

#### 3.2.1 倒排索引
倒排索引，首先得明白是什么东西倒了。

传统索引的映射是`文档Id -> 文档内容`

倒排索引的映射是`关键词 -> 文档Id`

So，很明显是key和value的映射关系倒过来了。一个倒排索引由文档中所有不重复词的列表构成，对于其中每个词，有一个包含它的文档列表。通过这样一张表，我们很容易根据关键词检索到对应的文档，并且多关键词时，还可以根据文档的关键词命中数量来进行匹配程度排名。

ES建立倒排索引的具体过程、示例以及如何区分关键词（大小写、单复数、近义词）等问题，可以点击[elastic.co 倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html)进入elastic官网了解，不在此详细展开了（主要是因为官网的文档写的太好了，实在没有复制粘贴过来的必要）。

#### 3.2.2 分析器

明白了什么是倒排索引后，我们来探索实现倒排索引的前提：文档内容被拆分成单独的词。

在ElasticSearch中，`analyzer（分析器）`专门用来分析（分词，规范化文本）文本数据，analyzer是一个包，这个包由三部分组成，分别是：`character filters （字符过滤器）`、`tokenizer（分词器）`、`token filters（token过滤器）`。

- ES内置分析器，并且支持自定义分析器。
- 一个analyzer可以有0个或多个character filters
- 一个analyzer有且只能有1个tokenizer
- 一个analyzer可以有0个或多个token filters

通过分其工作流程是：

- 首先，字符过滤器对被分析（analyzed）文本进行过滤和处理，例：从原始文本中移除HTML标记，根据字符映射替换文本等。
- 过滤之后的文本被分词器接收，分词器把文本分割成标记流，也就是一个接一个的标记。
- 然后，标记过滤器对标记流进行过滤处理，例如，移除停用词，把词转换成其词干形式，把词转换成其同义词等。
- 最终，过滤之后的标记流被存储在倒排索引中。
- ElasticSearch在收到用户的查询请求时，会使用分析器对查询条件进行分析，根据分析的结构，重新构造查询，以搜索倒排索引，完成全文搜索请求。

**什么时间使用分析器？**

 > 当我们`索引`一个文档，它的全文域被分析成词以用来创建倒排索引。 但是，当我们在全文域`搜索`的时候，我们需要将查询字符串通过相同的分析过程 ，以保证我们搜索的词条格式与索引中的词条格式一致。

### 3.3 记一次应用
#### 3.3.1 产品需求
摘自需求文档：
> 融360传数据给渠道：
第二天回传前一天该渠道的进件数据，字段包含，进件日期、渠道联合登录时传给我们的uid、申请产品名称、渠道号。
#### 3.3.2 遇到的问题
需求很明确，核心需求点就一个，`按进件渠道区分订单`。

但是好巧不巧，在订单表中进件渠道channel，不是一个单独的字段，而是以serialize序列化的形式存在订单表的extra
_info字段中。而orders表的extra_info长这个样子：
```
a:14:{s:8:"platform";s:1:"4";s:6:"source";s:0:"";s:6:"medium";s:0:"";s:8:"app_name";s:4:"xxxx";s:11:"app_subname";s:6:"xxxsdk";s:11:"app_version";s:5:"3.6.9";s:7:"channel";s:8:"sdk_xxxx;s:6:"is_xfd";i:0;s:30:"is_independent_shop_in_big_app";i:0;s:6:"app_id";N;s:11:"from_hhr_h5";N;s:5:"price";s:2:"21";s:11:"is_old_user";b:0;s:18:"vip_server_steward";i:0;}
```
一般情况下MySQL的like模糊查询的写法为（field已建立索引）：
```sql
SELECT `column` FROM `table` WHERE `field` like '%keyword%';
```
用like查询是可以解决按进件渠道区分订单的需求的，但是我们explain一下这条sql会发现，这条sql并未用到索引，而是`全表搜索`，orders表虽然分表了，但是数据量依旧很大，脚本在数据量很大的表中去跑大量的全表搜索的sql语句，显示是不合理的。

这样一个简单的需求，在仅仅使用MySQL的情况下，让我感到很无助。

#### 3.3.3 解决方案
MySQL不行，我们就换个方案，谁行用谁。

上面我们说到ElasticSearch的强项就是全文索引，Github上海量的数据就是用ES做全文索引的，那在订单表中模糊查询一个字段就更不在话下了。

淘金云的orders表在ES中有全量的同步数据，通过以下的查询语句，这个需求迎刃而解。
```php
$params = array(
	'size' => 5000, 
    'query' => array(
	    'bool' => array(
	        'filter' => array(
	            'range' => array(
	                'create_time' => array(
	                    'lt' => $endTime, //限制查询订单的时间范围
                        'gte' => $startTime
                    )
	            )
	        ),
	        'must' => array(
		        'match' => array(
		            'extra_info' => $name //模糊匹配渠道的名称
	            )
	        )
	    )
	),
    '_source' => array(
		'user_id', //要查询的字段
		'product_name'
    )
);
```

## 4. 准实时搜索
### 4.1 什么是准实时搜索
ElasticSearch被称为准实时搜索，原因是对ES的写入操作成功后，写入的数据需要1秒钟后才能被搜索到，因此ES搜索是准实时或者又称为准实时（Near real time）。 
### 4.2 准实时搜索原理
#### 4.2.1 数据写入过程

#### 4.2.2 缓存机制

## 5. 一些基本操作&注意事项



参考资料：
1. 《elasticsearch权威指南》 -- clinton gormley, zachary tong
2. 《从原理到应用，Elasticsearch详解》 -- TalkingData技术专栏