#生产环境配置
```
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#集群名称，默认是elasticsearch
cluster.name: es_prod
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#节点名称
node.name: docker_node1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#可以指定es的数据存储目录，默认存储在es_home/data目录下
path.data: /var/data/elasticsearch
#
# Path to log files:
#可以指定es的日志存储目录，默认存储在es_home/logs目录下
path.logs: /var/log/elasticsearch
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#锁定物理内存地址，防止elasticsearch内存被交换出去,也就是避免es使用swap交换分区
bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#为es设置ip绑定，默认是127.0.0.1，也就是默认只能通过127.0.0.1 或者localhost才能访问
network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#为es设置自定义端口，默认是9200
#在同一个服务器中启动多个es节点的话，默认监听的端口号会自动加1：例如：9200，9201，9202...
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#设置其他节点连接此节点的地址，如果不设置的话，则自动获取
network.publish_host: 172.16.16.179
#通过这个ip列表进行节点发现，组建集群
discovery.zen.ping.unicast.hosts: ["172.16.16.179:9300","172.16.16.179:9301","172.16.16.178:9302"]
#discovery.zen.ping_timeout: 60s
#
# Prevent the "split brain" by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#这个参数决定了在选主过程中需要有多少个节点通信，通过配置这个参数来防止集群脑裂现象 (集群总节点数量/2)+1
discovery.zen.minimum_master_nodes: 2
#
# For more information, consult the zen discovery module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:

#gateway.recover_after_nodes: 3
#预期的节点加入集群，就进行数据恢复处理
gateway.expected_nodes: 3
#如未达到预期的节点加入集群，需要等待的时间
gateway.recover_after_time: 1m
#一个集群中的N个节点启动后,才允许进行数据恢复处理，默认是1
gateway.recover_after_nodes: 2
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
node.master: true
node.data: true
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
xpack.ml.enabled: false
xpack.security.enabled: false
xpack.monitoring.enabled: false
xpack.graph.enabled: false
xpack.watcher.enabled: false
node.max_local_storage_nodes: 256
transport.tcp.port: 9300
transport.tcp.compress: true
action.auto_create_index: .security,.monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```

将日志和数据，文件夹权限为es的user
添加：
path.logs: /var/log/elasticsearch
path.data: /var/data/elasticsearch

拷贝配置文件config下文件到其他目录/usr/local/esconfig/
要拷贝以下文件：

```
cp -r elasticsearch.yml  /usr/local/esconfig/
cp -r jvm.options  /usr/local/esconfig/
cp -r log4j2.properties /usr/local/esconfig/
```
启动命令：
ES_PATH_CONF=/usr/local/esconfig/ ./bin/elasticsearch -d


#### 将es的bin加入环境变量PATH中
```
export ES_HOME=/usr/local/elasticsearch-6.2.2
export PATH=$ES_HOME/bin
```

执行source profile生效后启动
ES_PATH_CONF=/usr/local/esconfig/  elasticsearch -d



#### 集群重启优化
shard重新复制，移动，删除，再次移动的过程，会大量的耗费网络和磁盘资源。对于数据量庞大的集群来说，可能导致每次集群重启时，都有TB级别的数据无端移动，可能导致集群启动会耗费很长时间。

比如我本来有10个node,集群重启时，有5个node
1.复制其他5个node的shard到本地
2.此时上线其他5个node
3.复制到新上线的5个node，原来的5个node删除自己的shard

生产优化的配置:
gateway.expected_nodes: 3
gateway.recover_after_time: 1m
gateway.recover_after_nodes: 2
等待至少2个节点在线，然后等待最多1分钟，或者3个节点都在线，开始shard recovery恢复的过程

![](/assets/30.png)

#### es关闭

jps 

ps -ef|grep Elasticsearch

kill -SIGTERM 15516
 
