## 简单查询

1.全文搜索
```
GET /order_index/order_type/_search
```
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

3.如果只想返回文档的 _source 字段，不需要其他元数据

```
GET /order_index/order_type/1/_source

```
4.返回部分字段

```
GET /order_index/order_type/1/_source?orderId,orderNo
```

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


效果图：
图1
![](/assets/13.png)
图2
![](/assets/14.png)
图3
![](/assets/15.png)
图4
![](/assets/16.png)
图5
![](/assets/17.png)
图6
![](/assets/50.png)