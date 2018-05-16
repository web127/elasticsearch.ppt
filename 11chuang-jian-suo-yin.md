# 创建索引

1.创建索引  

  
ES的API组成结构：使用RESTful API风格来命名API
API基本格式：http://<ip>:<port>/<索引>/<类型>/<文档id> 常用HTTP动词
>常用HTTP动词：GET/PUT/POST/DELETE 
>>PUT 执行创建或修改，如确定document的ID时
>>POST一般用于创建和查询，如不确定document的ID，可以直接POST， ES可以自己生成不会发生碰撞的UUID
>>GET一般用于查询
>>DELETE用于删除




我们现在有一个业务需求就是存储订单数据，需要创建订单文档，如下操作：
>一个文档必须的三个元数据元素如下
_index文档在哪存放
_type文档表示的对象类别
_id文档唯一标识，id可以自己指定，也可以自动生成


#####假设构建了的表如下：
是否主键|字段名|字段描述|数据类型|分词
-|-|-|
是|orderId|订单ID|long|否
否|userId|用户ID|long|否
否|orderNo|订单号|string|否
否|userName|用户名|string|否
否|totalPrice|价格|float|否
否|address|收货地址|string|是
否|createTime|创建时间|data|否

如果不创建mapping，ES会根据文档的字段数据自动识别类型
这里需要创建自己定义的mapping
#####创建自定义mapping
![](/assets/7.png)
![](/assets/8.png)

1.建立order_index索引  
2.在order_index下建立order_type类型  
3.每个订单数据是一个文档

简单的创建索引方式，只能使用PUT创建

```\`
PUT /order_index/
```

简单的创建类型方式，只能使用POST,除非指定document的ID

```
POST /order_index/order_type/
```
![](/assets/4.png)


以上创建索引和类型可以通过一条命令完成
```
PUT /order_index/order_type/1
{
  "orderId":10001,
  "userId":10001,
  "orderNo":"a10001",
  "userName":"John",
  "totalPrice":50.00,
  "address":"江苏省南京市江宁区XXX",
  "createTime":"2018-05-16 14:08:00"
}
```
![](/assets/5.png)




