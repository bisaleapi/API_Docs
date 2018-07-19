# WebSocket API Reference

## 请求与订阅说明

### 访问地址

- 行情地址：暂无

### 心跳

websocket服务支持双向心跳，无论哪一方发送ping消息，例如：

```json
{"ping": 1526454367}
```

对方需回复

```json
{"pong": 1526454367}
```

`备注: ping/pong 消息成对存在，如果客户端发现服务器不回复pong消息，可重连服务器。当行情服务器发现客户端不回复			pong消息或者pong消息错误会断开连接`

### 请求类型

websocket服务同时支持查询、订阅、取消订阅操作，查询就是请求响应模式，只返回一次结果。订阅会不断接收到推动的数据。

###请求结构

websocket请求的基础字段如下

| 参数名  | 是否必须 | 类型   | 描述                                    | 取值          |
| ------- | -------- | ------ | --------------------------------------- | ------------- |
| action   | true     | string | 订阅，取消订阅，请求响应                | sub,unsub,req |
| function | true     | string | 消息类别                                | 见下面文档    |
| id      | true     | string | 请求的id,由客户端自己生成，响应原样返回 | 保证唯一性    |

```json
{
    "action":"req",
    "function":"market.eth_btc.kline.1min",
    "id":"0",
    "param":{
        "from":0,
        "to":0
    }
}
```



响应的字段如下

| 参数名  | 类型   | 描述                                           | 默认值 |
| ------- | ------ | ---------------------------------------------- | ------ |
| code    | int    | 返回码，20000:成功，其他有错误，详见错误码文档 |        |
| message | string | 响应消息，成功返回ok                           |        |
| id      | string | 客户端请求id                                   |        |
| function | string | 请求function,数据类型                           |        |
| ts      | int64  | 服务器时间                                     |        |
| data    | object | 返回数据                                       |        |

示例：

`正确请求返回`

```json
{
    "code":20000,
    "message":"ok",
    "id":"0",
    "function":"market.*.*.*",
    "ts":1526636776,
    "data":[...]
}
```

`错误请求返回`

```json
{
    "code":10003,
    "message":"不支持的function",
    "id":"0",
    "function":"market.*.*.*",
    "ts":1526636776,
    "data":[]
}
```

## 1. 请求

### 1.1 历史K线(kline)

除基础字段外，还需如下字段

| 参数名 | 是否必须 | 类型  | 描述     | 默认值               |
| ------ | -------- | ----- | -------- | -------------------- |
| from   | false    | int64 | 开始时间 | 当前服务器时刻       |
| to     | false    | int64 | 结束时间 | 当前服务器时刻减一天 |

function 字段格式如下:

```Json
market.$symbol.kline.$period
```

| 参数名 | 是否必须 | 类型   | 描述   | 取值                                               |
| ------ | -------- | ------ | ------ | -------------------------------------------------- |
| symbol | true     | string | 交易对 | etc_btc,bch_etc….                                  |
| period | true     | string | 周期   | 1min,5min,15min,30min,1hour,4hout,1day,1week,1year |

`请求示例`

```json
{
    "action":"req",
    "function":"market.eth_btc.kline.1min",
    "id":"0"
}
```

`响应`

```json
{
    "code":20000,
    "message":"ok",
    "id":"0",
    "function":"market.eth_btc.kline.1min",
    "ts":1526454367,
    "data":[
        {
            "time":1526443740,
            "open":0.0008,
            "high":0.0009,
            "low":0.0008,
            "close":0.0008,
            "amount":64.3,
            "volume":0.05364,
            "tick_count":25
        },
        {
            "time":1526443680,
            "open":0.0008,
            "high":0.0009,
            "low":0.0008,
            "close":0.0008,
            "amount":54.6,
            "volume":0.04526,
            "tick_count":25
        }
        .....
    ]
}
```

`备注: 为缩短服务器响应时间，一次请求最大返回300条K线数据`

### 1.2 交易明细(trade detail)

function 字段格式如下:

```json
market.$symbol.trade.detail
```

| 参数名 | 是否必须 | 类型   | 描述   | 取值              |
| ------ | -------- | ------ | ------ | ----------------- |
| symbol | true     | string | 交易对 | etc_btc,bch_etc…. |

`请求示例`

```json
{
    "action":"req",
    "function":"market.eth_btc.trade.detail",
    "id":"0"
}
```

`响应`

```json
{
    "code":20000,
    "message":"ok",
    "id":"0",
    "function":"market.eth_btc.trade.detail",
    "ts":1526525246,
    "data":[
        {
            "tid":"1878718",
            "amount":1.3,
            "price":0.0009,
            "timestamp":1526525246,
            "side":"1"
        },
        {
            "tid":"1878717",
            "amount":5,
            "price":0.0008,
            "timestamp":1526525246,
            "side":"2"
        }
        ...
    ]
}
```

### 1.3 获取市场深度全量(market depth)

function 字段格式如下:

```json
market.$symbol.depth.$type
```

| 参数名 | 是否必须 | 类型   | 描述     | 取值                                      |
| ------ | -------- | ------ | -------- | ----------------------------------------- |
| symbol | true     | string | 交易对   | etc_btc,bch_etc….                         |
| type   | true     | string | 深度类型 | depth0,depth1,depth2,depth3,depth4,depth5 |

`请求示例`

```json
{
    "action":"req",
    "function":"market.eth_btc.depth.depth0",
    "id":"0"
}
```

`响应`

```json
{
    "code":20000,
    "message":"ok",
    "id":"0",
    "function":"market.eth_btc.depth.depth0",
    "ts":1526528848,
    "data":{
        "version":20433,
        "data":{
            "sell":[
                [0.0008,1.2]
                ...
            ],
            "buy":[
                [0.0007, 0.9]
                ....
            ],
        }
    }
}
```

`备注:目前只支持depth0`

### 1.4 获取全市场行情(market.all.change.detail)

function 字段取值如下:

```json
market.all.change.detail
```

`请求示例`

```json
{
    "action":"req",
    "function":"market.all.change.detail",
    "id":"0"
}
```

`响应`

```json
{
    "code":20000,
    "message":"ok",
    "id":"0",
    "function":"market.all.change.detail",
    "ts":1526537733,
    "data":{
        "BTC":[
            {
                "symbol":"ETH_BTC",
                "price":0,
                "last_price":0,
                "high_price":0,
                "low_price":0,
                "amount":0,
                "volume":0,
                "period":"1day",
                "timestamp":0
            },
            {
                "symbol":"BCH_BTC",
                "price":0,
                "last_price":0,
                "high_price":0,
                "low_price":0,
                "amount":0,
                "volume":0,
                "period":"1day",
                "timestamp":0
            }
            ....
        ],
        "ETH":[
            {
                "symbol":"NDX_ETH",
                "price":0,
                "last_price":0,
                "high_price":0,
                "low_price":0,
                "amount":0,
                "volume":0,
                "period":"1day",
                "timestamp":0
            },
            {
                "symbol":"BCH_ETH",
                "price":0,
                "last_price":0,
                "high_price":0,
                "low_price":0,
                "amount":0,
                "volume":0,
                "period":"1day",
                "timestamp":0
            }
            ....
        ]
    }
}
```

## 2. 订阅

### 2.1 订阅K线(kline)

订阅K线数据，向服务器发送如下结构的数据：

```Json
{
    "action":"sub",
    "function":"market.$symbol.kline.$period",
    "id":"0"
}
```

| 参数名 | 是否必须 | 类型   | 描述   | 取值                                               |
| ------ | -------- | ------ | ------ | -------------------------------------------------- |
| symbol | true     | string | 交易对 | etc_btc,bch_etc….                                  |
| period | true     | string | 周期   | 1min,5min,15min,30min,1hour,4hout,1day,1week,1year |

`请求示例`

```json
{
    "action":"sub",
    "function":"market.eth_btc.kline.1min",
    "id":"0"
}
```

以后K线更新时推送

```json
{
    "function":"market.eth_btc.kline.1min",
    "ts":1526531833,
    "data":{
        "symbol":"ETH_BTC",
        "trading_time":1521530296,
        "open_price":9084,
        "high_price":9121,
        "low_price":8537,
        "close_price":8551,
        "amount":21,
        "volume":185071,
        "tick_count":6,
        "timestamp":1521530296017
    }
}
```

### 2.2 订阅交易明细(trade detail)

订阅交易明细，向服务器发送如下结构的数据：

```json
{
    "action":"sub",
    "function":"market.$symbol.trade.detail",
    "id":"0"
}
```

| 参数名 | 是否必须 | 类型   | 描述   | 取值              |
| ------ | -------- | ------ | ------ | ----------------- |
| symbol | true     | string | 交易对 | etc_btc,bch_etc…. |

`请求示例`

```json
{
    "action":"sub",
    "function":"market.eth_btc.trade.detail",
    "id":"0"
}
```

交易明细推送

```json
{
    "function":"market.eth_btc.trade.detail",
    "ts":1526550718622,
    "data":{
        "tid":1980928,
        "timestamp":1526550718622,
        "symbol":"ETH_BTC",
        "side":"1",
        "size":0.9,
        "price":0.0009
    }
}
```

### 2.3 订阅市场深度(market depth)

目前只支持depth0

订阅深度数据，向服务器发送如下结构的数据：

```Json
{
    "action":"sub",
    "function":"market.$symbol.depth.$type",
    "id":"0"
}
```

| 参数名 | 是否必须 | 类型   | 描述     | 取值                                      |
| ------ | -------- | ------ | -------- | ----------------------------------------- |
| symbol | true     | string | 交易对   | etc_btc,bch_etc….                         |
| type   | true     | string | 深度类型 | depth0,depth1,depth2,depth3,depth4,depth5 |

`请求示例`

```Json
{
    "action":"sub",
    "function":"market.eth_btc.depth.depth0",
    "id":"0"
}
```

`市场深度更新时推送`

```Json
{
    "function":"market.eth_btc.depth.depth0",
    "ts":1526610972794,
    "version":28,
    "data":{
        "sell":[
            [0.0009, 181]
            ...
        ],
        "buy":[
            [0.0008,85.1]
            ....
        ]
    }
}
```

### 2.4 订阅市场明细(market detail)

订阅市场实时数据，向服务器发送如下结构的数据：

```Json
{
    "action":"sub",
    "function":"market.$symbol.change.detail",
    "id":"0"
}
```

| 参数名 | 是否必须 | 类型   | 描述   | 取值              |
| ------ | -------- | ------ | ------ | ----------------- |
| symbol | true     | string | 交易对 | etc_btc,bch_etc…. |

`请求示例`

```json
{
    "action":"sub",
    "function":"market.eth_btc.change.detail",
    "id":"0"
}
```

`行情更新时推送`

```Json
{
    "function":"market.eth_btc.change.detail",
    "ts":1526611314638,
    "data":{
        "symbol":"ETH_BTC",
        "price":0.0008,
        "last_price":0.0009,
        "high_price":0.0009,
        "low_price":0.0008,
        "amount":58641.399999999325,
        "volume":49.860090000002295,
        "period":"1day",
        "rate":0,
        "timestamp":1526611314638
    }
}
```

## 3 取消订阅

```Json
{
    "action":"unsub",
    "function":"market.*.*.*",
    "id":"0"
}
```

