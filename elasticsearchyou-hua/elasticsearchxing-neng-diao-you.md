#ElasticSearch性能调优
1.慢查询或慢写入日志
2.查询优化建议
3.索引写入性能优化
4.搜索性能优化
5.磁盘读写性能优化


#### 1.慢查询或慢写入日志


针对query phase和fetch phase单独设置慢查询的阈值（你认为的慢）

在elasticsearch.yml中，设置
```
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms

index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug: 500ms
index.search.slowlog.threshold.fetch.trace: 200ms
```
慢查询日志格式设置
在log4j2.properties中配置的，比如配置：
```
appender.index_search_slowlog_rolling.type = RollingFile
appender.index_search_slowlog_rolling.name = index_search_slowlog_rolling
appender.index_search_slowlog_rolling.fileName = ${sys:es.logs}_index_search_slowlog.log
appender.index_search_slowlog_rolling.layout.type = PatternLayout
appender.index_search_slowlog_rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %.10000m%n
appender.index_search_slowlog_rolling.filePattern = ${sys:es.logs}_index_search_slowlog-%d{yyyy-MM-dd}.log
appender.index_search_slowlog_rolling.policies.type = Policies
appender.index_search_slowlog_rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.index_search_slowlog_rolling.policies.time.interval = 1
appender.index_search_slowlog_rolling.policies.time.modulate = true

logger.index_search_slowlog_rolling.name = index.search.slowlog
logger.index_search_slowlog_rolling.level = trace
logger.index_search_slowlog_rolling.appenderRef.index_search_slowlog_rolling.ref = index_search_slowlog_rolling
logger.index_search_slowlog_rolling.additivity = false
```


索引慢写入日志

可以用如下的配置来设置索引写入慢日志的阈值：
```
index.indexing.slowlog.threshold.index.warn: 10s
index.indexing.slowlog.threshold.index.info: 5s
index.indexing.slowlog.threshold.index.debug: 2s
index.indexing.slowlog.threshold.index.trace: 500ms
index.indexing.slowlog.level: info
index.indexing.slowlog.source: 1000
```
用下面的log4j.properties配置就可以设置索引慢写入日志的格式：
```
appender.index_indexing_slowlog_rolling.type = RollingFile
appender.index_indexing_slowlog_rolling.name = index_indexing_slowlog_rolling
appender.index_indexing_slowlog_rolling.fileName = ${sys:es.logs}_index_indexing_slowlog.log
appender.index_indexing_slowlog_rolling.layout.type = PatternLayout
appender.index_indexing_slowlog_rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %marker%.10000m%n
appender.index_indexing_slowlog_rolling.filePattern = ${sys:es.logs}_index_indexing_slowlog-%d{yyyy-MM-dd}.log
appender.index_indexing_slowlog_rolling.policies.type = Policies
appender.index_indexing_slowlog_rolling.policies.time.type = TimeBasedTriggeringPolicy
appender.index_indexing_slowlog_rolling.policies.time.interval = 1
appender.index_indexing_slowlog_rolling.policies.time.modulate = true

logger.index_indexing_slowlog.name = index.indexing.slowlog.index
logger.index_indexing_slowlog.level = trace
logger.index_indexing_slowlog.appenderRef.index_indexing_slowlog_rolling.ref = index_indexing_slowlog_rolling
logger.index_indexing_slowlog.additivity = false
```
#### 查询优化建议
1.搜索结果不要返回过大的结果集
2.document的内容不要超过100mb，http.max_context_length的默认值是100mb，es会拒绝写入。

#### 索引写入性能优化
1.用bulk批量写入
对单个es node的单个shard做压测，看看单线程最多一次性写多少条数据，性能是比较好的。
2、使用多线程将数据写入es
3、增加refresh间隔

默认的refresh间隔是1s，用index.refresh_interval参数可以设置，这样会其强迫es每秒中都将内存中的数据写入磁盘中，创建一个新的segment file。正是这个间隔，让我们每次写入数据后，1s以后才能看到。但是如果我们将这个间隔调大，比如30s，可以接受写入的数据30s后才看到，那么我们就可以获取更大的写入吞吐量，因为30s内都是写内存的，每隔30s才会创建一个segment file。
4、使用自动生成的id
如果我们要手动给es document设置一个id，那么es需要每次都去确认一下那个id是否存在，这个过程是比较耗费时间的。如果我们使用自动生成的id，那么es就可以跳过这个步骤，写入性能会更好。对于你的业务中的表id，可以作为es document的一个field。
5、index buffer
如果要进行非常重的高并发写入操作，那么最好将index buffer调大一些，indices.memory.index_buffer_size，这个可以调节大一些，设置的这个index buffer大小，是所有的shard公用的，但是如果除以shard数量以后，算出来平均每个shard可以使用的内存大小，一般建议，但是对于每个shard来说，最多给512mb，因为再大性能就没什么提升了。es会将这个设置作为每个shard共享的index buffer，那些特别活跃的shard会更多的使用这个buffer。默认这个参数的值是10%，也就是jvm heap的10%，如果我们给jvm heap分配10gb内存，那么这个index buffer就有1gb，对于两个shard共享来说，是足够的了。

