# 修改文档

#####修改整个文档，相当于覆盖

PUT或者POST请求

```
PUT /order_index/order_type/1
{
  "orderId":10001,
  "userId":10001,
  "orderNo":"a10001",
  "userName":"John",
  "totalPrice":50.00,
  "address":"银河系天蝎座XX号星球",
  "createTime":"2018-05-16 14:08:00"
}
```
![](/assets/11.png)

修改后可以看到_version变化
#####修改文档的部分数据

需要POST请求,带_update动词
```
http://172.16.16.179:9200/order_index/order_type/1/_update
{
  "doc": {
    "address": "银河系天蝎座XX号星球的，距离约三万光年"
  }
}
```
![](/assets/12.png)

