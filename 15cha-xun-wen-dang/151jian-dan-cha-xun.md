## 简单查询

1.全文搜索
```
GET /order_index/order_type/_search
```

2.根据_id查询，pretty表示以易读的格式返回
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
