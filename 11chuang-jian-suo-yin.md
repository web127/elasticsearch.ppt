#创建索引
1.创建索引
我们现在有一个业务需求就是存储订单数据，如下操作：
1.建立order_index索引
2.在order_index下建立order_type类型
3.每个订单是一个文档

最简单的方式
````
PUT /order_index/
```