
给order表添加关联字段


![](/assets/28.png)


如果有多个集合
```
{
  "doc": {
    "user": [
      {
        "userId": "2",
        "userName": "bbb",
        "mobile": "138******13",
        "age": "28",
        "sex": "M"
      },
      {
        "userId": "1",
        "userName": "aaa",
        "mobile": "138******12",
        "age": "27",
        "sex": "F"
      }
    ]
  }
}
```

查询结果
```
POST /order_index/order_type/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "orderNo": "a10001"
          }
        },
        {
          "match": {
            "user.userId": 1
          }
        },
        {
          "match": {
            "user.age": 27
          }
        }
      ]
    }
  }
}
```


![](/assets/29.png)