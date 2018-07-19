# 引擎Websocket-API接口说明文档

## WS线上地址

- wss://trade.bisale.com

## 整体概要介绍

- 客户端与引擎采用ws方式连接
- 客户端通过发送一个指定格式的json（其中分为：签名request和普通request）来与服务器交互。服务器在每次收到request之后，会返回相应的response来完成此次交互。
- 目前api涵盖功能如下
    1. SignedRequest
        1. 下单
        2. 撤单
        3. 查看账户余额
        4. 订阅账户资产信息变动
        5. 批量查看订单
    2. Request
        1. 订阅orderbook

## request概要介绍

request分为2种，1种是需要签名的request，简称SignedRequest（与账号相关操作，全部需要签名），另一种是普通request（用于订阅盘口信息等），简称Request

- Request，SignedRequest一起包含如下字段
    1. MsgType，用以区分这是什么请求
    2. CRID，用以来区分每个request，建议使用uuid（也叫做guid）

- SignedRequest独自包含如下字段
    1. Date，日期，精确到日，格式为（yyyyMMdd，例：20180415）
    2. Account，账户id（引擎内部的account，例：Uxxxx）
    3. SIG，用以存放将所有字段按照一定顺序，一定算法作出的摘要值。当SignedRequest 发送给引擎之后，引擎会按照同样方式去计算SIG的值。建议全大写，大小写不敏感

## SignedRequest 签名介绍

- 字段顺序介绍
    1. 所有的SignedRequest不管是下单还是撤单，前几个字段都是一致的，变化是在每个请求不同的后几个字段中。
    2. 前几个字段为（以下为顺序敏感）
        1. MsgType
        2. CRID
        3. Date
        4. Account

- 数字字段处理原则
    1. 如果一个price是0.1，在json中可以传0.1000（可以在有效位后多0），但是在计算签名的原始string中，必须是0.1。如果是1.0000，则必须为1（没有小数）（C#中，通过ToString(“0.############”)实现）

- 具体原理介绍
    1. 拿到password（通过调用getaccount api获取，如果没有，也可以向交易所小伙伴询问获取），用hash 算法（下面会详细贴代码介绍）对其进行计算 得到一个hash（用来做签名，具体要用到的key）
        1. Hash的具体算法
            1. 代码如下Convert.ToBase64String(m_SHA1Provider.ComputeHash(ASCIIEncoding.ASCII.GetBytes(txtHashSource.Text)))
        2. 说明
            1. 先把password用ascii码获取byte数组
            2. 然后使用sha1计算hash的byte数组
            3. 最后再用base64string将2中的byte数组变成string
        3. hash具体例子
            1. password
                1. test
            2. ascii码获取的byte数组
                1.   [0] 116 byte
                2.   [1] 101 byte
                3.   [2] 115 byte
                4.   [3] 116 byte
            3. hash的byte数组：
                1.   [0] 169 byte
                2.   [1] 74 byte
                3.   [2] 143 byte
                4.   [3] 229 byte
                5.   [4] 204 byte
                6.   [5] 177 byte
                7.   [6] 155 byte
                8.   [7] 166 byte
                9.   [8] 28 byte
                10.   [9] 76 byte
                11.   [10] 8 byte
                12.   [11] 115 byte
                13.   [12] 211 byte
                14.   [13] 145 byte
                15.   [14] 233 byte
                16.   [15] 135 byte
                17.   [16] 152 byte
                18.   [17] 47 byte
                19.   [18] 187 byte
                20.   [19] 211 byte
            4. base64string转换之后的string
                1. qUqP5cyxm6YcTAhz05Hph5gvu9M=
    2. 将所有字段按照特定顺序组合成一串string（之后会有每个请求的字段意义，以及签名顺序的文档）（同时数字类型的tostring，要稍微注意一下，下面有章节会介绍），然后用hash做hmac sha 1摘要签名，将结果放于SIG字段
        1. 签名的具体算法
            1. 按照特定顺序将SignedRequest中的所有字段拼出原始待签名string（即source string）
            2. 将hash用ascii 编码转换成 byte数组
            3. 用2中的byte数组创建hmacsha1对象（C#中如此，其余语言类似）
            4. 将source string 用ascii编码转换成byte数组，使用hmacsha1 的computeHash计算出byte数组
            5. 将4中的byte数组，通过tostring(“X2”)，方式转换成string，得到SIG字段
        2. 签名的具体例子
            1. source string
                1. PlaceOrderRequest150225516302420170809ptest1ETHCNY121.23100001000:00:00
            2. password
                1. test
            3. Hash
                1. qUqP5cyxm6YcTAhz05Hph5gvu9M=
            4. SIG
                1. C7B2A4A0126682B0125D68DABE2026DA367C57C6

## 报文格式介绍

### SignedRequest报文格式介绍

`需要加上前置相关字段，同时以下罗列字段为字段签名顺序`

SignedRequest 公共前置字段

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| MsgType | string | 消息类型 | "PlaceOrderRequest" |
| CRID   | string | 建议使用uuid，用以区分每个请求 | "639ae8e3-3a4b-40b2-acac-1f6160444d80" |
| Date | string | 当天日期 | "20180719" |
| Account | string | 账号 | "UNXXXX" |

SignedRequest 公共后置字段

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| SIG   | string | 签名摘要生成的结果 | "C7B2A4A0126682B0125D68DABE2026DA367C57C6" |

PlaceOrderRequest

`下单请求`

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| Symbol | string | 交易货币对 | "ETH_BTC" |
| Side   | string | 买卖方向，可用值1（买），2（卖） | "1", "2" |
| OrderType | string | 订单类型，可用值2（限价单） | "2" |
| Quantity | decimal | 数量（买到的货币单位） | 1.5 |
| Price   | decimal | 价格（报价货币，用什么币去买） | 2.5 |
| StopPrice | decimal | 止损价（目前没开止损单） | 0 |

```json
{"Symbol":"ETH_BTC","Side":"1","OrderType":"2","Quantity":1.0,"Price":1.0,"StopPrice":1500.0,"Date":"20180719","Account":"Utest1","SIG":"2DC1C9EEED7BB2A01C99428868D1AB28C37F3BEC","CRID":"a0092708-e840-4d86-b5e8-574a52f4e0d5","MsgType":"PlaceOrderRequest"}
```

CancelOrderRequest

`撤单`

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| Symbol | string | 交易货币对 | "ETH_BTC" |
| OID   | string | 订单id唯一的，是一串uuid | "C7B2A4A0126682B0125D68DABE2026DA367C57C6" |

```json
{"Symbol":"ETH_BTC","OID":"b59e81d533a649f5a4b0d3773773c5d0","Date":"20180719","Account":"Utest1","SIG":"2714613C8BA64875434D5C9FADF00BC0889E65AF","CRID":"88901f0930dc4f95b455e21cbb425fa2","MsgType":"CancelOrderRequest"}
```

GetAccountInfoRequest

`查询账号余额，及挂单汇总信息，无额外参数`

```json
{"Date":"20180719","Account":"Utest1","SIG":"B98D3EC1E9248D7E74BE185A00E5E1F7E51283D4","CRID":"45116b3080164a75912b12adf68c41e2","MsgType":"GetAccountInfoRequest"}
```

SubscribeRequest

`订阅账号资产变动信息，订阅后，挂单变化，资金变化全会推送，无额外参数`

```json
{"Date":"20180719","Account":"Utest1","SIG":"06000733C245FD25B02FBDBE9DFE7605CFE5F3D4","CRID":"799d1f97c4394775805a4bd250c7e42a","MsgType":"SubscribeRequest"}
```

GetOrdersRequest

`批量查询挂单信息，包含历史挂单（已完成或取消）`

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| Symbol | string | 交易货币对 | "ETH_BTC" |
| Begin  | long | timestamp 时间戳，utc，毫秒级，根据订单创建时间过滤 | 1531792846339 |
| End    | long | Timestamp 时间戳，utc，毫秒级，根据订单创建时间过滤，可以考虑超过当前时间一点，避免由于时间不一致漏单 | 1531792846339 |
| Status   | string | 过滤你需要的订单状态，可选值有A（pending，未到orderbook里面）0（new，在orderbook里）1（partialfill，部分成交）2（完全成交）3（交割完成）S（系统完成）4（已取消）G（系统取消）,建议将有效挂单和历史挂单分开请求，即 "A,0,1,2"和 "3,S,4,G" | "3,S,4,G"|

```json
{"Symbol":"ETH_BTC","Begin":1531994296120,"End":1531994296113,"Status":"","Type":null,"Date":"20180719","Account":"Utest1","SIG":"9373D2A72214909701D874995E924BA4D7A6C3EC","CRID":"a9c5fce534ec42fb85ab730ca82cfd05","MsgType":"GetOrdersRequest"}
```
            
### Request，无需签名

QuoteRequest

`获取并订阅orderbook信息`

| 参数名 | 类型 | 描述 | 取值例子 |
| - | - | - | - |
| Symbol | string | 交易货币对 | "ETH_BTC" |
| QuoteType | long | 订阅类型，2是全部订阅，0是只orderbook，1是只是ticker | 2 |

```json
{"Symbol":"ETH_BTC","QuoteType":0,"Count":1,"CRID":null,"MsgType":"QuoteRequest"}
```

### Response简介

`所有Response包含账户信息以及基础字段`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Date | string | 日期，精确到日，格式为（yyyyMMdd） | "20180415" |
| Account | string | 账户id（引擎内部的account） | "UXXXXXX" |
| RC | string | 错误原因代码 | "0" |
| CRID | string | 用以来区分每个request，建议使用uuid（也叫做guid） | "639ae8e3-3a4b-40b2-acac-1f6160444d80" |
| MsgType | string | 用以区分这是什么类型的响应 | "PlaceOrderResponse" |

### Response报文格式介绍

PlaceOrderResponse

`下单响应`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| OID | string | 订单的uuid（引擎生成） | "C7B2A4A0126682B0125D68DABE2026DA367C57C6" |
| OrdRejReason | int | 下单失败原因（1：未知的交易对； 109：数量超限； 110：保证金不足；） | 1 |

```json
{"OID":"2ca523049e9d471e87753b51bdf1b175","Text":null,"OrdRejReason":0,"Date":"20180717","Account":"Utest1","RC":"0","CRID":"954237a1-e335-414c-91de-89f668ca4903","MsgType":"PlaceOrderResponse"}
```

CancelOrderResponse

`撤单响应`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| OID | string | 订单的uuid（引擎生成） | "C7B2A4A0126682B0125D68DABE2026DA367C57C6" |
| OrdStatus | string | 订单状态 | "3" |
| CxlRejReason | int | 取消失败原因（0：订单已经被取消；） | 0 |

```json
{"OID":"2ca523049e9d471e87753b51bdf1b175","OrdStatus":"4","CxlRejReason":-1,"Date":"20180717","Account":"Utest1","RC":"0","CRID":"32332434cda64e67884d85c06af64059","MsgType":"CancelOrderResponse"}
```

GetAccountInfoResponse

`查询余额响应`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| AccountInfo | object | 账户信息，具体信息见Notification部分 |  |

```json
{"AccountInfo":{"RP":"Speculation","G":"dt","BL":[{"CR":"BTC","B":9997.0,"F":7.0},{"CR":"ETH","B":0.999,"F":0.0}],"CDL":[{"S":"BCH_BTC","TSS":0.0,"TBS":0.0,"IMF":1.0,"BIMR":0.0,"QIMR":0.0},{"S":"BCH_ETH","TSS":0.0,"TBS":0.0,"IMF":1.0,"BIMR":0.0,"QIMR":0.0},{"S":"ETH_BTC","TSS":0.0,"TBS":6.0,"IMF":1.0,"BIMR":0.0,"QIMR":7.0}],"Account":"Utest1","MsgType":"AccountInfo"},"Date":"20180717","Account":"Utest1","RC":"0","CRID":"4d25e6765611402b9f44291edb581545","MsgType":"GetAccountInfoResponse"}
```

SubscribeResponse

`订阅响应，无额外参数`

```json
{"Date":"20180717","Account":"Utest1","RC":"0","CRID":"a59f7fe1b5c14b94a26b5ccec779935e","MsgType":"SubscribeResponse"}
```

GetOrdersResponse

`批量查询挂单信息响应`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Orders | List(Order) | Order，具体信息见Notification部分 |  |
| OrdRejReason | int | 错误信息（112： 订单数量为0） | 112 |

```json
{"Orders":[{"Timestamp":1531792846339,"CRID":"639ae8e3-3a4b-40b2-acac-1f6160444d80","OID":"921719a87c8843a4a31bdfa7428891f1","Symbol":"ETH_BTC","Side":"2","OrderType":"2","LastQty":1.0,"CumQty":1.0,"LeaveQty":0.0,"TotalQty":1.0,"Price":2.0,"StopPrice":0.0,"Status":"3","Text":null,"SettlementQty":1.0,"AveragePrice":2.0,"ExecutionDetails":[{"Index":0,"Timestamp":1531792846276,"Price":2.0,"TotalQuantity":1.0,"OpenedQuantity":0.0,"ClosedQuantity":1.0}],"Created":1531792846260,"FeeExchange":0.0,"TradeFeeLogs":[],"Account":"ptest2","MsgType":"Order"}],"OrdRejReason":0,"Date":"20180717","Account":"ptest2","RC":"0","CRID":"e4ea07affa1a4105a0d205e85a5ad68f","MsgType":"GetOrdersResponse"}
```

### Notification简介

- 当有订阅资金账号变动时，会推相应的报文Order，AccountInfo等。

- 当有订阅OrderBook信息时，会推送orderbook

### Notification报文格式介绍

AccountInfo

`账号信息`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| BL | List(AccountBalance) | AccountBalance，具体信息见下表 |  |
| CDL | List(ContractDetail) | ContractDetail，具体信息见下表 |  |
| Account | string | 引擎账户 | "UXXXXX" |
| MsgType | string | 用以区分这是什么类型的推送 | "AccountInfo" |

AccountBalance

`资产信息`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| CR | string | 数字货币 | "BTC" |
| B | decimal | 总额 | 10000 |
| F | decimal | 总冻结金额 | 10000 |

ContractDetail

`交易对明细`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| S | string | 交易对 | "ETH_BTC" |
| TSS | decimal | 全部委托卖出数量 | 10000 |
| TBS | decimal | 全部委托购买数量 | 10000 |
| BIMR | decimal | 基础货币（ETH_BTC为例，ETH为基础货币）冻结 | 10000 |
| QIMR | decimal | 报价货币（ETH_BTC为例，BTC为报价货币）冻结 | 10000 |

```json
{"RP":"Speculation","G":"dt","BL":[{"CR":"BCH","B":10.0,"F":0.0},{"CR":"BTC","B":10000.0,"F":1.0}],"CDL":[{"S":"BCH_BTC","TSS":0.0,"TBS":0.0,"IMF":1.0,"BIMR":0.0,"QIMR":0.0},{"S":"BCH_ETH","TSS":0.0,"TBS":0.0,"IMF":1.0,"BIMR":0.0,"QIMR":0.0}],"Account":"ptest1","MsgType":"AccountInfo"}
```

Order

`订单信息`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Timestamp | long | 订单最近一次修改时间，时间戳，utc，毫秒级 | 1531792846339 |
| OID | string | 订单的uuid（引擎生成） | "C7B2A4A0126682B0125D68DABE2026DA367C57C6" |
| Symbol | string | 交易对 | "ETH_BTC" |
| Side | string | 值1（买），2（卖） | "1" |
| OrderType | string | 值2（限价单） | "2" |
| CumQty | decimal | 累计成交数量 | 10 |
| LeaveQty | decimal | 未成交数量 | 100 |
| TotalQty | decimal | 全部数量 | 110 |
| Price | decimal | 价格 | 6300 |
| StopPrice | decimal | 止损价格 | 0 |
| Status | string | 订单状态（在request部分已说明） | "3" |
| AveragePrice | decimal | 平均价格 | 55 |
| ExecutionDetails | List(ExecutionDetail) | ExecutionDetail，具体信息见下表 |  |
| Created | long | 订单创建时间，时间戳，utc，毫秒级 | 1531792846260 |
| FeeExchange | decimal | 手续费（推送order无手续费） | 0.5 |
| TradeFeeLogs | List(TradeFeeLog) | TradeFeeLog，具体信息见下表 |  |
| Account | string | 引擎账户 | "UXXXXXX" |
| MsgType | string | 用以区分这是什么类型的推送 | "Order" |

ExecutionDetail

`成交明细`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Index | int | 序号 | 1 |
| Timestamp | long | 时间戳，utc，毫秒级 | 1531792846339 |
| Price | decimal | 价格 | 6300 |
| TotalQuantity | decimal | 全部数量 | 110 |

TradeFeeLog

`成交手续费记录`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Created | long | 手续费收取时间，时间戳，utc，毫秒级 | 1531792846260 |
| FeeType | string | 手续费类型 | "Exchange" |
| FeeSubType | string | 手续费子类型 | "Maker" |
| Quantity | decimal | 成交数量 | 10 |
| FeeTotal | decimal | 手续费金额 | 1.1 |
| Currency | string | 手续费货币 | "ETH" |

```json
{"Timestamp":1531792846339,"CRID":"639ae8e3-3a4b-40b2-acac-1f6160444d80","OID":"921719a87c8843a4a31bdfa7428891f1","Symbol":"ETH_BTC","Side":"2","OrderType":"2","LastQty":1.0,"CumQty":1.0,"LeaveQty":0.0,"TotalQty":1.0,"Price":2.0,"StopPrice":0.0,"Status":"3","Text":null,"SettlementQty":1.0,"AveragePrice":2.0,"ExecutionDetails":[{"Index":0,"Timestamp":1531792846276,"Price":2.0,"TotalQuantity":1.0,"OpenedQuantity":0.0,"ClosedQuantity":1.0}],"Created":1531792846260,"FeeExchange":0.0,"TradeFeeLogs":[],"Account":"ptest2","MsgType":"Order"}
```

OrderBook

`盘口信息，当收到增量以后用增量中的orderbookdata,以价格为key，更新本地原有orderbook，某个价格数量为0时，需要在本地移除对应价格的orderbookdata`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Timestamp | long | 时间戳，utc，毫秒级 | 1531792846339 |
| Symbol | string | 交易对 | "ETH_BTC" |
| Version | int | 版本，用于确认是否有消息丢失，version号从 0 到 32766 之间循环，每次新消息version+1，如果发现差值不是1，则漏包，需要重拉  | 1 |
| Type | string | 推送类型（I：增量，L：全量），目前，推送的orderbook全部type为I，在QuoteResponse中，orderbook为L | "I" |
| List | List(OrderBookData) | OrderBookData，具体信息见下表 |  |
| MsgType | string | 用以区分这是什么类型的推送 | "OrderBook" |

OrderBookData

`包含买方与卖方的具体数据`

| 字段名 | 类型 | 描述 | 例子 |
| - | - | - | - |
| Side | string | 值1（买），2（卖） | "1" |
| Size | decimal | 数量 | 10 |
| Price | decimal | 价格 | 6300 |

```json
{"Timestamp":1531965515958,"Symbol":"ETH_BTC","Version":1,"Type":"I","Content":"L","List":[{"Side":"1","Size":1.0,"Price":1.0}],"MsgType":"OrderBook"}
```
