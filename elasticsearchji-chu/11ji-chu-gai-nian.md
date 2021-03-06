# 1.1基础概念

##### 集群和节点：

> 一个集群是由一个或多个ES组成的集合,每一个集群都有一个唯一的名字  
> 每一个节点都有自己的名字,每一个节点都是通过集群的名字来加入集群的，节点能够存储数据，参与集群索引数据以及搜索数据的独立服务

##### 概念：

> Index（索引）相当于SQL里的DataBase，也就是数据库   
> Type（类型）相当于SQL里的Table，也就是表，在6.0版本已经弃用这个概念
  官方不再建议在索引中创建多个类型。并在后续高版本弃用type，详细见官方文档。   
> Document（文档）相当于SQL里的一行记录，也就是一行数据  
> Field（字段）就是相当于SQL里的一个字段  
> Shard（分片）单台机器无法存储大量数据，es可以将一个索引中的数据切分为多个shard，分布在多台服务器上存储。有了shard就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。每个shard都是一个lucene index。  
> Replica shard（副本分片）replica可以在shard故障时提供备用服务，保证数据不丢失，多个replica还可以提升搜索操作的吞吐量和性能。  
> Mapping（映射）它定义了索引中每个字段类型，以及索引的其他设置，可事先定义，也可以根据第一次存储的文档自动识别，类似mysql里建表时对字段定义数据类型。

###### 建立索引前手动自定义分片：

```
{
  "settings": {
    "number_of_shards": "3",
    "number_of_replicas": "1",
    "refresh_interval": "30s"
  }
}
```

![](/assets/1.png)

* primary shard默认为5个，并且一旦建好不能修改，replica shard默认1个，随时修改数量

* 一般使用默认的分片就可以了，就是5个primary shard，每个primary shard拥有一个replica shard。也就是说每个索引有10个分片

####修改副本shard数量

```
PUT   index/_settings
{
  "number_of_replicas": "2",
  "refresh_interval": "30s"
}
```

![](/assets/41.png)


###### Mapping的创建:

Mapping一旦创建，字段类型不能修改，只能增加字段，类似于mysql，因此建表前需要考虑好。

![](/assets/2.png)

使用Head插件查看索引信息![](/assets/3.png)

