## 简单查询

1.全文搜索
```
GET /order_index/order_type/_search
```
![](/assets/13.png)

```
POST /order_index/order_type/_search
{
  "query": {
    "match_all": {}
  }
}
```
2.根据_id查询，pretty表示以易读的格式返回
```
GET /order_index/order_type/1
```

```
GET /order_index/order_type/1?pretty
```

![](/assets/14.png)


3.如果只想返回文档的 _source 字段，不需要其他元数据

```
GET /order_index/order_type/1/_source

```
![](/assets/15.png)


4.返回部分字段

```
GET /order_index/order_type/1/_source?orderId,orderNo
```
![](/assets/16.png)

5.根据字段查询， 通过URL参数来传递查询信息
```
GET /order_index/order_type/_search?q=orderId:10001
```
传JSON体参数，POST请求
```
POST /order_index/order_type/_search
{
    "query": {
	 "match": {
	   "orderId": 10001
	  }
      }
}
```
![](/assets/17.png)


6.返回从10到19的文档,注意:如果没有使用size字段, 默认为10
```
POST /order_index/order_type/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

7.字段orderId降序排列
```
POST /order_index/order_type/_search
{
  "query": {
    "match_all": {}
  },
  "sort": {
    "orderId": {
      "order": "desc"
    },
    "userId": {
      "order": "desc"
    }
  }
}
```

8.mget查询

```
POST _mget
{
  "docs": [
    {
      "_index": "order_index",
      "_type": "order_type",
      "_id": 1
    },
    {
      "_index": "order_index",
      "_type": "order_type",
      "_id": 2
    }
  ]
}
```

![](/assets/50.png)

9.使用_all字段查询

_all字段是把所有其它字段中的值，以空格为分隔符组成一个大字符串，_all能让你在不知道要查找的内容是属于哪个具体字段的情况下进行搜索
query_string 和 simple_query_string 查询默认都是查询 _all 字段

```
POST /order_index/order_type/_search
{
  "query": {
      "query_string": {
      "query": "鼓楼"
    }
  }
}
```
等同于：
```
GET /order_index/order_type/_search?q=鼓楼
```
![](/assets/53.png)