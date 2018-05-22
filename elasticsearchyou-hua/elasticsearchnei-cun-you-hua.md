1.内存
es生产环境中是很吃内存的，吃的机器的内存，jvm heap（堆内存）还是用的比较少的
如果一个机器有64G的内存，那么是比较理想的状态，但是32GB和16GB的内存也是ok的。
尽量不要小于8G。
2.CPU
选择性能较为一般有更多的cpu core。因为更多的cpu core可以提供更强的并发处理能力，远比单个cpu性能高带来的效果更加明显。
3、磁盘
如果我们能够使用SSD固态硬盘，而不是机械硬盘，那么当然是最好的。
4、网络
对于es这种分布式系统来说，快速而且可靠的网络是非常的重要的。
5、JVM
5.x版本以上建议使用JDK1.8
官方推荐，绝对不要随便调整jvm的设置。虽然jvm有几百个配置选项，而且我们可以手动调优jvm的几乎方方面面。
6.容量规划
es要达到ms级，必须要有足够的os cache去缓存大部分的索引数据，一般来说，除非是你的机器的内存资源，完全可以容纳所有的落地的磁盘文件的os cache，ms，否则的话，如果不是的话，会大量走磁盘，秒级
比如我们的订单数据总数大概30G，三台机器内存16G的总内存为48G，这样只剩下18G，系统用8G左右，只有10G做磁盘索引文件的读写，这样大概就是秒级，要达到ms级三台机器内存要达到70G以上。


这里主要讲内存

不要随意调节jvm和设置threadpool参数
es默认用的垃圾回收器是CMS。CMS回收器是并发式的回收器，能够跟应用程序工作线程并发工作，最大程度减少垃圾回收时的服务停顿时间。
es不要去调节默认的jvm，对于es来说，大部分的参数都保留为默认的就可以了。
在es中，默认的threadpool设置是非常合理的，大多数的磁盘IO操作都是由lucene的线程管理的，而不是由es管理的，因此es的线程不需要关心这个问题。

#### 1、jvm heap分配
es默认会给jvm heap分配2个G的大小，对于几乎所有的生产环境来说，这个内存都太小了。如果用这个默认的heap size，在生产环境的集群肯定表现不会太好。
调节有两个方式
1.启动时
比如：ES_JAVA_OPTS="-Xms10g -Xmx10g" ./bin/elasticsearch，但是要注意-Xms和-Xmx最小和最大堆内存一定设置的一样，避免运行过程中jvm扩容，会消耗时间。
2.
es 5.x以上，一般推荐在jvm.options文件里面去设置jvm相关的参数。

#### 2、设置多大内存给es
将机器上少于一半的内存分配给es，但是还有个觉得es读写搜索的用户lucene，是要使用底层的os filesystem cache来缓存数据结构，es的性能很大的一块，其实是由有多少内存留给操作系统的os cache，供lucene去缓存索引文件，来决定的。所以说lucene的os cache有多少是非常重要的。
一般建议的是，将50%的内存分配给es jvm heap，然后留50%的内存给os cache。
#### 3、不要超过32G内存
不要将超过32G内存的内存分配给es的jvm heap
如果heap小于32G的化，jvm会用一种技术来压缩对象的指针，会自动采用32位pointer，如果你给jvm heap分配超过32G的内存，实际上是没有什么意义的，因为用64位的pointer，1/3的内存都给object pointer给占据了，超过32G,就没法用32位pointer。不用32位pointer，就只能用64位pointer，此时object pinter的大小会急剧增长，更多的cpu到内存的带宽会被占据，更多的内存被耗费。

可以给jvm option加入-XX:+PrintFlagsFinal，然后可以打印出来UseCompressedOops是否为true。这就可以让我们找到最佳的内存设置。可以不断调节内存大小，然后观察是否启用compressed oops。
#### 系统配置优化
在生产环境中下面的一些设置必须配置一下：
（1）资源限制
（2）禁止swapping
（3）确保拥有足够的虚拟内存
（4）确保拥有足够的线程数量
（5）允许es有最大虚拟内存大小的检查
在/etc/security/limits.conf中
1.资源限制

```
#esuser soft nofile 65536
esuser hard nofile 65536
```

2.swapping（交换）
大多数操作系统都会使用尽量多的内存来进行file system cache，并且尽量将不经常使用的java应用的内存swap到磁盘中去。这会导致jvm heap的部分内存，甚至是用来执行代码的内存页被swap到磁盘中去。性能会有多差。
因此通常建议彻底关闭机器上的swap
禁止的方式：
1）临时性禁止swap：swapoff -a
2）要永久性的禁止swap，需要修改/etc/fstab文件，然后将所有包含swap的行都注释掉
3）配置swappiness，通过sysctl，将vm.swappiness设置为1，sysctl -w vm.swappiness=1
4）启用bootstrap.memory_lock
将es jvm进程的address space锁定在内存中，阻止es内存被swap out到磁盘上去。
在config/elasticsearch.yml中，可以配置：

bootstrap.memory_lock: true
启动会报锁定内存失败错误，需要设置/etc/security/limits.conf权限
![](/assets/31.png)

```
#esuser soft memlock unlimited
esuser hard memlock unlimited

```
查看配置是否成功：

```
GET _nodes?filter_path=**.mlockall
```

![](/assets/32.png)

3.确保拥有足够的虚拟内存
修改/etc/sysctl.conf，将vm.max_map_count的值修改一下
```
vm.max_map_count=262144
```
4.确保拥有足够的线程数量
在/etc/security/limits.conf中设置nproc为2048来确保es用户能创建的最大线程数量至少在2048以上。

```
#esuser soft nproc 2048
esuser hard nproc 2048
```
5.允许es有最大虚拟内存大小的检查

```
#esuser soft as unlimited
esuser hard as unlimited

```
