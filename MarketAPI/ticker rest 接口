## 一、全局变量
   host:服务域名，不同环境不同（测试https://api01.bisale.org;生产https://api.bisale.com）  
## 二：接口信息  
 
### 1. 获取交易对相关信息  
 
调用URL：

```
GET {host}/api/quoteRequest
```

请求参数：  

字段名|类型|是否必填|描述|例子
---|---|:---:|---|---|
symbol|String|否|交易对|“ELF_BTC”

示例：
```
curl -X GET {host}/api/quoteRequest?symbol=ELF_BTC
```

返回格式：  

字段名|类型|描述
---|---|---
code|int|状态码
data|Object|返回数据，是一个由json格式数据组成的集合。  
返回值相应字段：  

字段名|类型|描述|例子
---|---|---|---
Symbol|String|交易对|“ELF_BTC”
BidPrice|String|出价|“8.682”
AskPrice|String|报价|"8.7333“
Open|String|开盘价|"0.0"
High|String|最高价|"9.9576"
High24H|String|24小时最高价|"0.0"
Low|String|最低价|"8.2612"
Low24H|String|24小时最低价|"0.0"
Rate24H|String|24小时差价|"-15.2421887915358"
Last|String|上一笔成交价|"0.00005127"
Volume|String|成交量|"81182.3021"
Volume24H|String|24小时成交量|"301895.5912"
Timestamp|Long|推送时间|"1534314516174"

返回值如：  

````json
{
  "code": 200,
  "data": [
      {
          "Symbol": "IONC_BTC",
          "BidPrice": "0.0",
          "AskPrice": "0.0",
          "Open": "0.0",
          "High": "0.0",
          "High24H": "0.0",
          "Low": "0.0",
          "Low24H": "0.0",
          "Rate24H": "0.0",
          "Last": "0.0",
          "Volume": "0.0",
          "Volume24H": "0.0",
          "Timestamp": 1534291202496
      },
      {
          "Symbol": "BCH_BTC",
          "BidPrice": "0.0827044",
          "AskPrice": "0.08320997",
          "Open": "0.0",
          "High": "0.08480417",
          "High24H": "0.0864134",
          "Low": "0.08235806",
          "Low24H": "0.07944227",
          "Rate24H": "-3.91376800357352",
          "Last": "0.08303138",
          "Volume": "2.84",
          "Volume24H": "11.8744",
          "Timestamp": 1534314516346
      }
  ]
}
````  

### 2.新增交易对信息

调用URL:

```
GET {host}/api/sendQuoteRequest
```

请求参数：  

字段名|类型|是否必填|描述|例子
---|---|:---:|---|---|
symbol|String|是|交易对|“ELF_BTC”
auth|String|是|密钥|”abcdrwasdfa123fafsf“  

示例：

```
curl -X GET {host}/api/sendQuoteRequest?symbol=ELF_BTC&auth=abcdrwasdfa123fafsf  
```

返回格式：  

字段名|类型|描述
---|---|---
code|int|状态码
data|Object|返回数据，无
如： 
 
````json
{
  "code": 200,
  "data": {}
}
````
