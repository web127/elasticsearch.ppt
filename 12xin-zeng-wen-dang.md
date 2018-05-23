# 新增文档

按上一篇内容，我们创建多个文档
```
PUT /order_index/order_type/2
{
  "orderId":10002,
  "userId":10002,
  "orderNo":"a10002",
  "userName":"Jane",
  "totalPrice":80.00,
  "address":"江苏省南京市鼓楼区XXX",
  "createTime":"2018-05-16 14:30:00"
}
```
```
PUT /order_index/order_type/3
{
  "orderId":10003,
  "userId":10003,
  "orderNo":"a10003",
  "userName":"Douglas",
  "totalPrice":100.00,
  "address":"江苏省南京市玄武区XXX",
  "createTime":"2018-05-16 14:33:00"
}
```

#### 使用bulk批量写入数据
bulk api

```
POST /order_index/order_type/_bulk
{ "index" : { "_index" : "test", "_type" : "_doc", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "_doc", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "_doc", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "_doc", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```



