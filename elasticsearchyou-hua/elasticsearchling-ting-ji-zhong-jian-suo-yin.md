# ElasticSearch零停机重建索引

基于scroll+bulk+索引别名实现零停机重建索引

一个field的设置后市不能被修改的，如果要修改，那么需要重新按新的mapping，建立一个index

实例操作
![](/assets/58.png)


假如现在我要将createTime改为date类型，现在是字符串

![](/assets/60.png)

直接修改是不行的，因为此时createTime字段已经建立，又不能直接删除index重建
![](/assets/59.png)


不停机修改方式：


1.对索引建立别名

```
PUT /order_index/_alias/bm_order_index/
```

![](/assets/61.png)

java代码里改为对别名的bm_order_index继续进行操作

2.新建新的索引order_index_new

![](/assets/62.png)

![](/assets/63.png)


3.使用scroll将数据一批批的查询出来

```
POST /order_index/order_type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": [
    "_doc"
  ],
  "size": 1
}
```
![](/assets/64.png)

4.使用bulk将查询出来的数据批量导入新索引

