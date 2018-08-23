
## 一、全局变量
   host:服务域名，不同环境不同（测试https://api03.bisale.org;生产https://api.bisale.com）  
## 二：接口信息  
 
### 1. 获取成交记录 
 
调用URL：

```
GET {host}/api/getTradesRequest
```

请求参数：  

字段名|类型|是否必填|描述|例子
---|---|:---:|---|---|
begin |long|是|开始时间|1530602654000
end|long|是|结束时间|1535959454000
symbol|String|是|交易对|“ELF_BTC”
pageSize|int|否|数据条目数(不传采用默认值)|50

示例：
```
curl -X GET {host}/api/getTradesRequest?begin=1530602654000&end=1535959454000&symbol=ETH_BTC&pageSize=10
```

返回格式：  

字段名|类型|描述
---|---|---
code|int|状态码
data|Object|返回数据，是一个由json格式数据组成的集合。

返回值相应字段：  

字段名|类型|描述|例子
---|---|---|---
Timestamp|long|成交时间|1530602654234
Symbol|String|交易对|"ELF_BTC"
Side|String|买卖方向，1:买，2:卖|"1"
Size|bigDecimal|成交数量|7.3
Price|bigDecimal|成交价格|0.0009

返回值如：  

````json
{
    "code": 200,
    "data": [
        {
            "Timestamp": 1530602654234,
            "Symbol": "ETH_BTC",
            "Side": "1",
            "Size": 7.3,
            "Price": 0.0009
        },
        {
            "Timestamp": 1530602654000,
            "Symbol": "ETH_BTC",
            "Side": "1",
            "Size": 1.1,
            "Price": 0.0009
        }
    ]
}
````  
