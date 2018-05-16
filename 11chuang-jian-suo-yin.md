# 创建索引

1.创建索引  

  
ES的API组成结构：使用RESTful API风格来命名API
API基本格式：http://<ip>:<port>/<索引>/<类型>/<文档id> 常用HTTP动词
>常用HTTP动词：GET/PUT/POST/DELETE 
>>POST一般用于创建和查询，如不确定document的ID，可以直接POST， ES可以自己生成不会发生碰撞的UUID
>>PUT 执行创建或修改，如确定document的ID时
>>DELETE用于删除
>>GET一般用于查询



我们现在有一个业务需求就是存储订单数据，需要创建订单文档，如下操作：
>一个文档必须的三个元数据元素如下
_index文档在哪存放
_type文档表示的对象类别
_id文档唯一标识，id可以自己指定，也可以自动生成

1.建立order\_index索引  
2.在order\_index下建立order\_type类型  
3.每个订单数据是一个文档

最简单的创建索引方式

```\`
PUT /order_index/
```

最简单的创建类型方式



