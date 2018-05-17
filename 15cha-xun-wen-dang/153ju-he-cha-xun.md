## 聚合查询

#### 求和

Sum求和
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "return_totalPrice": {
      "sum": {
        "field": "totalPrice"
      }
    }
  }
}
```
![](/assets/22.png)





#### 平均
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "return_avg_totalPrice": {
      "avg": {
        "field": "totalPrice"
      }
    }
  }
}
```
![](/assets/25.png)


#### 最大最小
max、min
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "return_min_totalPrice": {
      "min": {
        "field": "totalPrice"
      }
    }
  }
}
```
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "return_max_totalPrice": {
      "max": {
        "field": "totalPrice"
      }
    }
  }
}
```
![](/assets/23.png)
![](/assets/24.png)

#### 统计查询
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "return_stats_totalPrice": {
      "stats": {
        "field": "totalPrice"
      }
    }
  }
}
```
![](/assets/26.png)

#### 分组
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "group_by_totalPrice": {
      "terms": {
        "field": "totalPrice"
      }
    }
  }
}
```
![](/assets/27.png)



#### 多条件聚合

比如按totalPrice升序，按orderId分组
```
POST /order_index/order_type/_search
{
  "size": 0,
  "aggs": {
    "totalPrice": {
      "terms": {
        "field": "totalPrice",
        "order": {
          "_count": "asc"
        }
      }
    },
    "group_by_totalPrice": {
      "terms": {
        "field": "orderId"
      }
    }
  }
}
```

