## 简单查询

1.查询所有
```
GET /order_index/order_type/_search
```

2.根据_id查询
```
GET /order_index/order_type/1
```

3.如果只想返回文档的 _source 字段，不需要其他元数据

```
GET /order_index/order_type/1/_source
```

4.

