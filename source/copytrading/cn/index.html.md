---
title: BTSE API Documentation
language_tabs:
  - json
toc_footers: []
includes: []
search: true
highlight_theme: darkula
code_clipboard: true
headingLevel: 2

---

# 更新日志

## 版本 1.0.0（2025年4月22日）

* 发布 跟单交易 API 的初始版本

# 概览

## 生成 API 密钥

在使用经过身份验证的 API 之前，您需要在 BTSE 平台上创建一个 API 密钥。要创建 API 密钥，您可以按照以下步骤操作：

* 使用您的用户名/电子邮件和密码登录 BTSE 网站
* 单击右上角的“帐户”
* 选择 API 选项卡
* 单击“新 API” 按钮以创建 API 密钥和密码（注意：密码仅会显示一次）
* 使用您的 API 密钥和密码构建签名。

## 端点

* 生产环境
  * HTTP
     * `https://api.btse.com/copytrading`
  * Websocket
     * `wss://ws.btse.com/ws/futures`
* 测试网络
  * HTTP
     * `https://testapi.btse.io/copytrading`
  * Websocket
     * `wss://testws.btse.io/ws/futures`
 
## 身份验证

 * API密钥（request-api）
   * 参数名称：`request-api`，位置：标头。API密钥以字符串形式从BTSE平台获取

 * API密钥（request-nonce）
   * 参数名称：`request-nonce`，位置：标头。当前时间戳的长格式表示

 * API密钥（request-sign）
   * 参数名称：`request-sign`，位置：标头。基于以下算法生成的复合签名：Signature=HMAC.Sha384 (secretkey, (urlpath + request-nonce + bodyStr))（注意：当没有数据时，bodyStr = ''）：

### 示例：下订单

> **HMAC SHA384 Signature**

```shell
$ echo -n "/api/v1/order1624985375123{\"postOnly\":false,\"price\":8500.0,\"reduceOnly\":false,\"side\":\"BUY\",\"size\":1,\"stopPrice\":0.0,\"symbol\":\"BTC-PERP\",\"time_in_force\":\"GTC\",\"trailValue\":0.0,\"triggerPrice\":0.0,\"txType\":\"LIMIT\",\"type\":\"LIMIT\"}" | openssl dgst -sha384 -hmac "848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx"
(stdin)= 943adfce43b609a28506274976b96e08cf4bdc4ea53ca0b4cac0eb2cf0773a7d0807efc0aeab779d47fadcd9a60eea13
```

* 下订单的端点是 `https://api.btse.com/futures/api/v1/order`
* 假设我们有以下值：
  * request-nonce: `1624985375123`
  * request-api: `4e9536c79f0fdd72bf04f2430982d3f61d9d76c996f0175bbba470d69d59816x`
  * secret: `848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx`
  * Path: `/api/v1/order`
  * Body: `{"postOnly":false,"price":8500.0,"reduceOnly":false,"side":"BUY","size":1,"stopPrice":0.0,"symbol":"BTC-PERP","time_in_force":"GTC","trailValue":0.0,"triggerPrice":0.0,"txType":"LIMIT","type":"LIMIT"}`
  * Encrypted Text: `/api/v1/order1624985375123{"postOnly":false,"price":8500.0,"reduceOnly":false,"side":"BUY","size":1,"stopPrice":0.0,"symbol":"BTC-PERP","time_in_force":"GTC","trailValue":0.0,"triggerPrice":0.0,"txType":"LIMIT","type":"LIMIT"}`
* 生成的签名将是：
  * request-sign: `943adfce43b609a28506274976b96e08cf4bdc4ea53ca0b4cac0eb2cf0773a7d0807efc0aeab779d47fadcd9a60eea13`


## 速率限制

* 强制执行以下速率限制：

BTSE 的速率限制如下：

**查询**

* 每个API：每秒 `15次请求`
* 每个用户：每秒 `30次请求`

**订单**

* 每个API：每秒 `75次请求`
* 每个用户：每秒 `75次请求`

### 机制描述

我们的系统实现了一个分层封锁机制，有三个不同的封锁时长：1秒、5分钟 和 15分钟。封锁时长的计算从第一次封锁开始时算起。
此外，如果 IP 地址或用户在 1 小时内或 15 分钟的封锁时长结束后没有超过速率限制，封锁时长的计算将被重置。

在返回 429 响应时，会包含一个 Retry-After 头部，并提供解锁的时间戳。

#### 速率限制等级

* 1秒
* 5分钟
* 15分钟

## API状态代码

每个API将返回以下HTTP状态之一：

* 200 - API请求成功，参考特定API响应以获取预期的有效负载
* 400 - 错误请求。服务器不会处理此请求。通常是因为请求中包含了无效的参数
* 401 - 未经授权的请求。服务器不会处理此请求，因为它没有有效的身份验证凭据
* 403 - 禁止的请求。提供了凭据，但它们不足以执行请求
* 404 - 未找到。表示服务器理解请求但无法找到目标资源的正确表示
* 405 - 不允许的方法。表示请求方法未被请求的服务器知道
* 408 - 请求超时。表示服务器未完成请求。BTSE API的超时设置为30秒
* 429 - 请求过多。表示客户端已超过服务器设置的速率限制。有关更多详细信息，请参阅速率限制
* 451 - 基于法律原因不可用。表示客户端因异常行为而被禁止
* 500 - 服务器内部错误。表示服务器遇到意外情况，无法满足请求

## API枚举

在连接到BTSE API时，您将遇到代表BTSE中不同状态或状态类型的数字代码。以下部分提供了您预计会看到的代码列表。

* 1: MARKET_UNAVAILABLE = 期货市场不可用
* 2: ORDER_INSERTED = 订单已成功插入
* 4: ORDER_FULLY_TRANSACTED = 订单已完全交易
* 5: ORDER_PARTIALLY_TRANSACTED = 订单已部分交易
* 6: ORDER_CANCELLED = 订单已成功取消
* 7: ORDER_REFUNDED = 订单已退款
* 8: INSUFFICIENT_BALANCE = 账户余额不足
* 9: TRIGGER_INSERTED = 触发订单已成功插入
* 10: TRIGGER_ACTIVATED = 触发订单已成功激活
* 11: ERROR_INVALID_CURRENCY = 无效货币错误
* 12: ERROR_UPDATE_RISK_LIMIT = 更新风险限额时出现错误
* 13: ERROR_INVALID_LEVERAGE = 无效杠杆错误
* 15: ORDER_REJECTED = 订单被拒绝
* 16: ORDER_NOTFOUND = 未找到订单，使用提供的订单ID或clOrderID
* 17: REQUEST_FAILED = 未能完成请求，请检查订单状态
* 20: SUCCESS = 操作成功
* 21: FREEZE_SUCCESSFUL = 冻结成功
* 27: TRANSFER_SUCCESSFUL = 期货和现货之间的资金转移成功
* 28: TRANSFER_UNSUCCESSFUL = 现货和期货之间的资金转移失败
* 29: QUERY_GET_ORDERS = 查询获取订单
* 31: QUERY_GET_POSITIONS = 查询获取持仓
* 33: QUERY_GET_ALL_POSITIONS_ORDERS = 查询获取所有持仓订单
* 34: QUERY_WALLET = 查询钱包
* 36: QUERY_FUTURES_MARGIN = 查询期货保证金
* 41: ERROR_INVALID_RISK_LIMIT = 指定了无效的风险限额
* 51: QUERY_GET_ORDERS_WITH_LIMIT = 查询获取带有限制的订单
* 64: STATUS_LIQUIDATION = 帐户正在清算
* 65: STATUS_ACITVE = 订单处于活动状态
* 66: MODE_BUY = 购买模式
* 76: ORDER_TYPE_LIMIT = 限价订单
* 77: ORDER_TYPE_MARKET = 市价订单
* 80: ORDER_TYPE_PEG = 挂单/算法订单
* 81: ORDER_TYPE_OTC = 场外交易订单
* 83: MODE_SELL = 卖出模式
* 85: STATUS_PROCESSING = 订单处于非活动状态
* 88: STATUS_INACTIVE = 订单处于非活动状态
* 101: FUTURES_ORDER_PRICE_OUTSIDE_LIQUIDATION_PRICE = 期货订单超出了清算价格
* 110: FUTURES_FUNDING = 期货资金
* 123: AMEND_ORDER = 订单已修改
* 124: UNFREEZE_SUCCESSFUL = 解冻成功
* 129: FUTURES_CONFIG_MODE_CHANGE = 期货仓位模式更改
* 131: FUTURES_STATUS_PROCESSING_LEVERAGE = 期货杠杆更改中
* 132: FUTURES_STATUS_PROCESSING_RISK_LIMIT = 风险限额更改中
* 133: FUTURES_POSITION_MODE_INVALID = 期货仓位模式错误
* 134: POSITION_MODE_UNCHANGEABLE = 无法更改期货仓位模式
* 138: POSITION_MODE_CHANGE_PROCESSING = 期货仓位模式更改中
* 300: ERROR_MAX_ORDER_SIZE_EXCEEDED = 超过最大订单大小错误
* 301: ERROR_INVALID_ORDER_SIZE = 无效订单大小错误
* 302: ERROR_INVALID_ORDER_PRICE = 无效订单价格错误
* 303: ERROR_RATE_LIMITS_EXCEEDED = 超过速率限制错误
* 304: ERROR_MAX_OPEN_ORDER_EXCEEDED = 超过最大开放订单数错误
* 305: ERROR_ORDER_PRICE_OUT_OF_PRICE_PROTECTION_RANGE = 价格超过开放订单范围
* 1003: ORDER_LIQUIDATION = 订单正在进行清算
* 1004: ORDER_ADL = 订单正在进行ADL
* 30410: BLOCK_TRADE_COMPLETE_SUCCESS = 区块交易已成功完成

## 垃圾订单

垃圾订单是指大量的小订单大小。为了确保平台和用户的利益不受恶意用户的侵害，我们将对下列情况的用户采取以下措施，这些用户下单小额订单。

[垃圾订单检测机制 : BTSE Support](https://support.btse.com/en/support/solutions/articles/43000720904-spam-order-detection-mechanism)

* 订单的名义价值低于5美元的将被标记为垃圾订单，并自动变为隐藏订单。
* 被标记为垃圾的订单始终支付吃单费。
* 被标记为垃圾的Post-Only API订单将被拒绝而不是被隐藏。
* 太多的垃圾订单可能导致暂时封禁交易账户。
* 放置 >= 4 个挂单，总大小小于 20 美元的API账户有可能被标记为垃圾账户。
* 被标记为垃圾的账户可能会对账户施加限制，包括订单速率限制、持仓限制，或禁用API功能。如对新的垃圾订单机制有疑问，请发送电子邮件至 mm@btse.com。

# 交易端点

## 创建新订单

> 请求（创建`市价`订单）

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "side": "BUY",
  "type": "MARKET"
}
```
> 请求（创建`限价`订单）

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "price": 21000,
  "side": "BUY",
  "type": "LIMIT"
}
```
> 请求（创建`限价` `触发` 订单）

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "price": 21000,
  "side": "BUY",
  "type": "LIMIT",
  "txType": "TRIGGER",
  "triggerPrice": 30000
}
```
> 请求（创建`限价` `止损` 订单）

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "price": 21000,
  "side": "BUY",
  "type": "LIMIT",
  "txType": "STOP",
  "triggerPrice": 30000
}
```
> 请求（创建 `OCO` 订单）

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "price": 21000,
  "side": "BUY",
  "type": "OCO",
  "txType": "LIMIT",
  "trigger": "markPrice",
  "stopPrice": 30010,
  "triggerPrice": 30000
}
```
> 请求 (创建`限价`订单并设置`止盈/止损（TP/SL）`)

```json
{
    "symbol": "BTC-PERP",
    "size": 10,
    "price": 29000,
    "side": "BUY",
    "type": "LIMIT",
    "takeProfitPrice": 31000,
    "takeProfitTrigger": "markPrice",
    "stopLossPrice": 27000,
    "stopLossTrigger": "lastPrice"
}
```
> 请求（仅使用`TP`创建`限价`订单）

```json
{
    "symbol": "BTC-PERP",
    "size": 10,
    "price": 29000,
    "side": "BUY",
    "type": "LIMIT",
    "takeProfitPrice": 31000,
    "takeProfitTrigger": "markPrice"
}
```

> 请求（仅使用`SL`创建`限价`订单）

```json
{
    "symbol": "BTC-PERP",
    "size": 10,
    "price": 29000,
    "side": "BUY",
    "type": "LIMIT",
    "stopLossPrice": 27000,
    "stopLossTrigger": "lastPrice"
}
```

> 响应（通用）

```json
[
  {
    "status": 4,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 21000.0,
    "side": "BUY",
    "size": 1,
    "orderID": "abb3f457-fdc0-4bdb-a46b-8e4aa49a57c2",
    "timestamp": 1660558270207,
    "triggerPrice": 0.0,
    "trigger": false,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 21000.0,
    "fillSize": 1.0,
    "clOrderID": "",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 0.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  }
]
```

> 响应（用于 `OCO` 订单）

```json
[
  {
    "status": 9,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 23000.0,
    "side": "BUY",
    "size": 1,
    "orderID": "4c9d16c1-9869-4734-bfb8-56318e961ef2",
    "timestamp": 1660558185243,
    "triggerPrice": 30000.0,
    "trigger": true,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 0.0,
    "fillSize": 0.0,
    "clOrderID": "",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 1.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  },
  {
    "status": 2,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 21000.0,
    "side": "BUY",
    "size": 1,
    "orderID": "53749446-39d3-4b72-87c9-92e9fc7e4b8c",
    "timestamp": 1660558185225,
    "triggerPrice": 0.0,
    "trigger": false,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 0.0,
    "fillSize": 0.0,
    "clOrderID": "",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 1.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  }
]
```

`POST /api/v1/order`

创建一个新的订单。需要 `Lead Trader` 权限

### 请求参数

| 名称           | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                                                                                                                     |
| ---           | ---     | ---      | ---                                                                                                                                                                                                                                                                                                                                                                     |
| symbol        | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                                                                                                                 |
| price         | double  | No       | 除非创建市场订单，否则为必填。订单价格                                                                                                                                                                                                                                                                                                                                  |
| size          | long    | Yes      | 订单尺寸以`合同大小`表示（即使在风险限额调整后也保持不变）                                                                                                                                                                                                                                                                                                               |
| side          | string  | Yes      | `BUY` 或 `SELL`                                                                                                                                                                                                                                                                                                                                                        |
| time_in_force | string  | No       | 订单的时间有效性<br/>GTC: 有效直至取消<br/>IOC: 立即或取消<br/>FOK: 全部成交或取消<br/>HALFMIN: 订单有效30秒<br/>FIVEMIN: 订单有效5分钟<br/>HOUR: 订单有效一个小时<br/>TWELVEHOUR: 订单有效12小时<br/>DAY: 订单有效一天<br/>WEEK: 订单有效一周<br/>MONTH: 订单有效一个月                                                                                                              |
| type          | string  | Yes      | 订单类型<br/>LIMIT: 限价订单<br/>MARKET: 市价订单<br/>OCO: 一个取消另一个                                                                                                                                                                                                                                                                                               |
| txType        | string  | No       | 用于停止订单或触发订单<br/>STOP: 停止订单，`triggerPrice` 是必填项<br/>TRIGGER: 触发订单，`triggerPrice` 是必填项<br/>LIMIT: 默认值，当其既不是停止订单也不是触发订单时使用                                                                                                                                                                                              |
| stopPrice     | double  | No       | 创建OCO订单时为必填。表示停止价格                                                                                                                                                                                                                                                                                                                                      |
| triggerPrice  | double  | No       | 创建停止、触发、OCO订单时为必填。表示触发价格                                                                                                                                                                                                                                                                                                                          |
| trailValue    | double  | No       | 跟踪值                                                                                                                                                                                                                                                                                                                                                                  |
| postOnly      | boolean | No       | 布尔值，表示这是否只做Maker(Post only) 订单，交易者将支付Maker手续费                                                                                                                                                                                                                                                                                                  |
| reduceOnly    | boolean | No       | 布尔值，将这笔订单设置为只减仓, 在双向持仓时，买方`BUY`减少空头仓位，卖方`SELL`则减少多头仓位                                                                                                                                                                                                                                                                                                                                       |
| clOrderID     | string  | No       | 自定义订单ID                                                                                                                                                                                                                                                                                                                                                            |
| trigger       | string  | No       | 用于创建txType: `STOP` 或 `TRIGGER` 的订单。有效选项: `markPrice` (默认) 或 `lastPrice`  |
| takeProfitPrice  | double  | No       | 在创建带有止盈订单时强制执行。指示触发价格
| takeProfitTrigger  | string  | No       | 用于创建带有止盈订单的订单。有效选项：`标记价格`（默认）或`最新价格`|
| stopLossPrice  | double  | No       | 在创建带有止损订单时强制执行。指示触发价格
| stopLossTrigger  | string  | No       | 用于创建带有止损订单的订单。有效选项：`标记价格`（默认）或`最新价格`|
| positionMode  | string  | No       | 用于创建指定仓位模式订单。有效选项：单向持仓`ONE_WAY`（默认|

### 响应内容

| 名称            | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                                                |
| ---           | ---     | ---      | ---                                                                                                                                                                                                                                                                                                |
| symbol        | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                                           |
| clOrderID     | string  | Yes      | 交易者发送的客户标签                                                                                                                                                                                                                                                                               |
| fillSize      | number  | Yes      | 已成交的交易大小                                                                                                                                                                                                                                                                                   |
| orderID       | string  | Yes      | 订单ID                                                                                                                                                                                                                                                                                             |
| orderType     | integer | Yes      | 订单类型 <br/>76: 限价订单<br/>77: 市价订单<br/>80: Algo订单                                                                                                                                                                                                                                       |
| postOnly      | boolean | Yes      | 表明订单是否为只做Maker(Post only) 订单                                                                                                                                                                                                                                                                           |
| price         | double  | Yes      | 订单价格                                                                                                                                                                                                                                                                                           |
| side          | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                                                                           |
| size          | long    | Yes      | 订单大小以`合同大小`表示（即使在风险限额调整后也保持不变）                                                                                                                                                                                                                                          |
| status        | long    | Yes      | 订单状态<br/>2: 订单已插入<br/>3: 订单已交易<br/>4: 订单已完全交易<br/>5: 订单部分交易<br/>6: 订单已取消<br/>7: 订单已退款<br/>9: 触发已插入<br>10: 触发已激活<br/>15: 订单被拒绝<br/>16: 订单未找到<br/>17: 请求失败                                                                                        |
| time_in_force | string  | Yes      | 订单有效性                                                                                                                                                                                                                                                                                         |
| timestamp     | long    | Yes      | 订单时间戳                                                                                                                                                                                                                                                                                         |
| trigger       | boolean | Yes      | 如果订单是触发订单的指示器                                                                                                                                                                                                                                                                        |
| triggerPrice  | double  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                                                                                                                                                                                                                                          |
| avgFillPrice  | double  | Yes      | 平均成交价格。对于部分交易的订单返回平均成交价格                                                                                                                                                                                                                                                  |
| message       | string  | Yes      | 交易消息                                                                                                                                                                                                                                                                                           |
| stealth       | double  | Yes      | 仅对Algo订单有效                                                                                                                                                                                                                                                                                   |
| deviation     | double  | Yes      | 仅对Algo订单有效                                                                                                                                                                                                                                                                                   |
| remainingSize | double  | Yes      | 剩余待交易的大小                                                                                                                                                                                                                                                                                   |
| originalSize  | double  | Yes      | 原始订单大小                                                                                                                                                                                                                                                                                       |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 创建新的算法订单

> 请求

```json
{
  "symbol": "BTC-PERP",
  "price": 21500,
  "size": 1,
  "side": "BUY",
  "clOrderID": "60a30188-f2a2-4498-b061-7d72126c18c2",
  "stealth": 10,
  "deviation": -10
}
```

> 响应

```json
[
  {
    "status": 2,
    "symbol": "BTC-PERP",
    "orderType": 80,
    "price": 21500.0,
    "side": "BUY",
    "size": 1,
    "orderID": "de9f94bb-0ca0-470b-830e-9bc2e109c719",
    "timestamp": 1660554373317,
    "triggerPrice": 0.0,
    "trigger": false,
    "deviation": -10.0,
    "stealth": 10.0,
    "message": "",
    "avgFillPrice": 0.0,
    "fillSize": 0.0,
    "clOrderID": "60a30188-f2a2-4498-b061-7d72126c18c2",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 1.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  }
]
```

`POST /api/v1/order/peg`

创建新的算法订单。算法订单是一种价格会根据市场价格变化的订单。要创建算法订单，用户需要输入额外的参数：

* `price`：用户愿意将订单列出的最低价格（卖单）或最高价格（买单）
* `deviation`：订单价格与指数价格的偏差程度。该值以百分比表示，范围从 `-10` 到 `10`
* `stealth`：订单簿上要显示多少百分比的订单量。

此API需要具有`交易`权限

### 请求参数

| 名称       | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                            |
| ---       | ---     | ---      | ---                                                                                                                                                                                                                                            |
| symbol    | string  | Yes      | 市场符号                                                                                                                                                                                                                                       |
| price     | double  | Yes      | 卖单的最低价，这是用户愿意出售的最低价格。买单的最高价，这是用户愿意购买的最高价格。                                                                                                                                                        |
| size      | long    | Yes      | 订单尺寸                                                                                                                                                                                                                                       |
| side      | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                       |
| clOrderID | string  | No       | 自定义订单ID                                                                                                                                                                                                                                    |
| deviation | double  | No       | 订单价格应与指数价格偏离多少。该值以百分比表示，范围从`-10`到`10`                                                                                                                                                                             |
| stealth   | double  | No       | 订单中应在订单簿上显示的百分比是多少。                                                                                                                                                                                                        |
| positionMode  | string  | No       | 用于创建指定仓位模式订单。有效选项：单向持仓`ONE_WAY`（默认）                                                                                                                                                                                                                                                          |

### 响应内容

| 名称            | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                                                |
| ---           | ---     | ---      | ---                                                                                                                                                                                                                                                                                                |
| symbol        | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                                           |
| clOrderID     | string  | Yes      | 交易者发送的客户标签                                                                                                                                                                                                                                                                               |
| fillSize      | number  | Yes      | 已成交的交易大小                                                                                                                                                                                                                                                                                   |
| orderID       | string  | Yes      | 订单ID                                                                                                                                                                                                                                                                                             |
| orderType     | integer | Yes      | 订单类型 <br/>76: 限价订单<br/>77: 市价订单<br/>80: Algo订单                                                                                                                                                                                                                                       |
| postOnly      | boolean | Yes      | 表明订单是否为只做Maker(Post only) 订单                                                                                                                                                                                                                                                                           |
| price         | double  | Yes      | 订单价格                                                                                                                                                                                                                                                                                           |
| side          | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                                                                           |
| size          | long    | Yes      | 订单大小以`合同大小`表示（即使在风险限额调整后也保持不变）                                                                                                                                                                                                                                          |
| status        | long    | Yes      | 订单状态<br/>2: 订单已插入<br/>3: 订单已交易<br/>4: 订单已完全交易<br/>5: 订单部分交易<br/>6: 订单已取消<br/>7: 订单已退款<br/>9: 触发已插入<br>10: 触发已激活<br/>15: 订单被拒绝<br/>16: 订单未找到<br/>17: 请求失败                                                                                        |
| time_in_force | string  | Yes      | 订单有效性                                                                                                                                                                                                                                                                                         |
| timestamp     | long    | Yes      | 订单时间戳                                                                                                                                                                                                                                                                                         |
| trigger       | boolean | Yes      | 如果订单是触发订单的指示器                                                                                                                                                                                                                                                                        |
| triggerPrice  | double  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                                                                                                                                                                                                                                          |
| avgFillPrice  | double  | Yes      | 平均成交价格。对于部分交易的订单返回平均成交价格                                                                                                                                                                                                                                                  |
| message       | string  | Yes      | 交易消息                                                                                                                                                                                                                                                                                           |
| stealth       | double  | Yes      | 订单的隐秘值                                                                                                                                                                                                                                                                                       |
| deviation     | double  | Yes      | 订单的偏差值                                                                                                                                                                                                                                                                                       |
| remainingSize | double  | Yes      | 剩余待交易的大小                                                                                                                                                                                                                                                                                   |
| originalSize  | double  | Yes      | 原始订单大小                                                                                                                                                                                                                                                                                       |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 查询订单

> 响应

```json
{
    "orderType": 76,
    "price": 1,
    "size": 111,
    "side": "BUY",
    "filledSize": 0,
    "orderValue": 0.111,
    "pegPriceMin": 0,
    "pegPriceMax": 0,
    "pegPriceDeviation": 1,
    "timestamp": 1698757024617,
    "orderID": "<Order UUID>",
    "stealth": 1,
    "triggerOrder": false,
    "triggered": false,
    "triggerPrice": 0,
    "triggerOriginalPrice": 0,
    "triggerOrderType": 0,
    "triggerTrailingStopDeviation": 0,
    "triggerStopPrice": 0,
    "symbol": "BTC-PERP",
    "trailValue": 0,
    "remainingSize": 111,
    "clOrderID": "<Order clOrderID>",
    "reduceOnly": false,
    "status": 2,
    "triggerUseLastPrice": false,
    "avgFilledPrice": 0,
    "timeInForce": "GTC",
    "takeProfitOrder": null,
    "stopLossOrder": null,
    "closeOrder": false,
    "contractSize": 0.001
}
```

## 修改订单

> 请求（修改价格）

```json
{
  "symbol": "BTC-PERP",
  "orderID": "604c3ebf-d7fa-468d-9ff0-f6ad030221b4",
  "type": "PRICE",
  "value": 22000
}
```

> 请求（全部修改）

```json
{
  "symbol": "BTC-PERP",
  "orderID": "604c3ebf-d7fa-468d-9ff0-f6ad030221b4",
  "type": "ALL",
  "orderPrice": 30010,
  "orderSize": 1,
  "triggerPrice": 30000
}
```

> 响应

```json
[
  {
    "status": 123,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 20000.0,
    "side": "BUY",
    "size": 1,
    "orderID": "604c3ebf-d7fa-468d-9ff0-f6ad030221b4",
    "timestamp": 1660639762254,
    "triggerPrice": 0.0,
    "trigger": true,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 0.0,
    "fillSize": 0.0,
    "clOrderID": "",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 1.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  }
]
```

`PUT /api/v1/order`

修改订单的价格、数量或触发价格。对于触发订单，如果订单已经被触发，触发价格将无法进一步修改。修订订单不适用于算法订单。

### 请求参数

| 名称          | 类型    | 是否必须 | 描述                                                                                                                                                                                                       |
| ---          | ---     | ---      | ---                                                                                                                                                                                                       |
| symbol       | string  | Yes      | 市场符号                                                                                                                                                                                                  |
| orderID      | string  | No       | 内部订单ID。当未提供`clOrderID`时为必填项。如果提供了`orderID`，将忽略`clOrderID`。                                                                                                                        |
| clOrderID    | string  | No       | 自定义订单ID。当未提供`orderID`时为必填项。                                                                                                                                                               |
| type         | string  | Yes      | 修改类型<br/>`PRICE`: 修改订单价格<br/>`SIZE`: 修改订单尺寸<br/>`TRIGGERPRICE`: 修改触发价格<br/>`ALL`: 修改多个字段                                                                                     |
| value        | number  | Yes      | 要修改的值。其值取决于设置的类型。                                                                                                                                                                       |
| orderPrice   | number  | No       | 对于类型：`ALL`，要修改的订单价格                                                                                                                                                                        |
| orderSize    | number  | No       | 对于类型：`ALL`，要修改的合同大小订单尺寸                                                                                                                                                                |
| triggerPrice | number  | No       | 对于类型：`ALL`，要修改的触发价格                                                                                                                                                                        |


### 响应内容

| 名称            | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                                                |
| ---           | ---     | ---      | ---                                                                                                                                                                                                                                                                                                |
| symbol        | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                                           |
| clOrderID     | string  | Yes      | 交易者发送的客户标签                                                                                                                                                                                                                                                                               |
| fillSize      | string  | Yes      | 已成交的交易大小                                                                                                                                                                                                                                                                                   |
| orderID       | string  | Yes      | 订单ID                                                                                                                                                                                                                                                                                             |
| orderType     | integer | Yes      | 订单类型 <br/>76: 限价订单<br/>77: 市价订单<br/>80: Algo订单                                                                                                                                                                                                                                       |
| postOnly      | boolean | Yes      | 表明订单是否为只做Maker(Post only) 订单                                                                                                                                                                                                                                                                           |
| price         | double  | Yes      | 订单价格                                                                                                                                                                                                                                                                                           |
| side          | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                                                                           |
| size          | long    | Yes      | 订单大小以`合同大小`表示（即使在风险限额调整后也保持不变）                                                                                                                                                                                                                                          |
| status        | long    | Yes      | 订单状态<br/>2: 订单已插入<br/>3: 订单已交易<br/>4: 订单已完全交易<br/>5: 订单部分交易<br/>6: 订单已取消<br/>7: 订单已退款<br/>9: 触发已插入<br>10: 触发已激活<br/>15: 订单被拒绝<br/>16: 订单未找到<br/>17: 请求失败                                                                                        |
| time_in_force | string  | Yes      | 订单有效性                                                                                                                                                                                                                                                                                         |
| timestamp     | long    | Yes      | 订单时间戳                                                                                                                                                                                                                                                                                         |
| trigger       | string  | Yes      | 如果订单是触发订单的指示器                                                                                                                                                                                                                                                                        |
| triggerPrice  | string  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                                                                                                                                                                                                                                          |
| avgFillPrice  | string  | Yes      | 平均成交价格。对于部分交易的订单返回平均成交价格                                                                                                                                                                                                                                                  |
| message       | string  | Yes      | 交易消息                                                                                                                                                                                                                                                                                           |
| stealth       | double  | Yes      | 订单的隐秘值                                                                                                                                                                                                                                                                                       |
| deviation     | string  | Yes      | 订单的偏差值                                                                                                                                                                                                                                                                                       |
| remainingSize | double  | Yes      | 剩余待交易的大小                                                                                                                                                                                                                                                                                   |
| originalSize  | double  | Yes      | 原始订单大小                                                                                                                                                                                                                                                                                       |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 取消订单

> 请求 (取消单个订单)

```
/api/v1/order?symbol=BTC-USD&clOrderID=my-order-id
```

> 响应

```json
[
  {
    "status": 6,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 19000.0,
    "side": "BUY",
    "size": 1,
    "orderID": "ae5b1b27-d5fe-41e2-89f8-f17b60fb3def",
    "timestamp": 1660640879996,
    "triggerPrice": 0.0,
    "trigger": false,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 0.0,
    "fillSize": 0.0,
    "clOrderID": "string",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 1.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "time_in_force": "GTC"
  }
]
```

`DELETE /api/v1/order`

取消尚未成交的待定订单。orderID是取消特定订单的唯一标识符。clOrderID是交易者发送的自定义ID。通过clOrderID取消时，所有具有相同ID的订单都将被取消。如果未发送orderID和clOrderID，则取消将针对当前市场中的所有订单。

### 请求参数

| 名称       | 类型    | 是否必须 | 描述                                                                                                                                                                    |
| ---        | ---     | ---      | ---                                                                                                                                                                    |
| symbol     | string  | Yes      | 市场符号                                                                                                                                                               |
| orderID    | string  | No       | 订单的唯一标识符。当未提供`clOrderID`时为必填项。如果提供了`orderID`，将忽略`clOrderID`。                                                                                |
| clOrderID  | string  | No       | 客户端自定义订单ID。当未提供`orderID`时为必填项。                                                                                                                                                  |


### 响应内容

| 名称            | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                                                        |
| ---             | ---     | ---      | ---                                                                                                                                                                                                                                                                                                        |
| symbol          | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                                                   |
| clOrderID       | string  | Yes      | 交易者发送的客户标签                                                                                                                                                                                                                                                                                       |
| fillSize        | double  | Yes      | 已成交的交易大小                                                                                                                                                                                                                                                                                           |
| orderID         | string  | Yes      | 订单ID                                                                                                                                                                                                                                                                                                     |
| orderType       | integer | Yes      | 订单类型 <br/>76: 限价订单<br/>77: 市价订单<br/>80: 算法订单                                                                                                                                                                                                                                               |
| postOnly        | boolean | Yes      | 表明订单是否为只做Maker(Post only) 订单                                                                                                                                                                                                                                                                                   |
| price           | double  | Yes      | 订单价格                                                                                                                                                                                                                                                                                                   |
| side            | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                                                                                   |
| size            | long    | Yes      | 以`合同大小`表示的订单大小（即使在风险限额调整后也保持不变）                                                                                                                                                                                                                                                  |
| status          | long    | Yes      | 订单状态<br/>2: 订单已插入<br/>3: 订单已交易<br/>4: 订单已完全交易<br/>5: 订单部分交易<br/>6: 订单已取消<br/>7: 订单已退款<br/>9: 触发已插入<br>10: 触发已激活<br/>15: 订单被拒绝<br/>16: 订单未找到<br/>17: 请求失败                                                                                              |
| time_in_force   | string  | Yes      | 订单有效性                                                                                                                                                                                                                                                                                                |
| timestamp       | long    | Yes      | 订单时间戳                                                                                                                                                                                                                                                                                                |
| trigger         | boolean | Yes      | 表明订单是否为触发订单的指示器                                                                                                                                                                                                                                                                            |
| triggerPrice    | double  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                                                                                                                                                                                                                                                  |
| avgFillPrice    | double  | Yes      | 平均成交价格。对于部分交易的订单返回平均成交价格                                                                                                                                                                                                                                                          |
| message         | string  | Yes      | 交易消息                                                                                                                                                                                                                                                                                                  |
| stealth         | double  | Yes      | 订单的隐秘值                                                                                                                                                                                                                                                                                              |
| deviation       | double  | Yes      | 订单的偏差值                                                                                                                                                                                                                                                                                              |
| remainingSize   | double  | Yes      | 剩余待交易的大小                                                                                                                                                                                                                                                                                          |
| originalSize    | double  | Yes      | 原始订单大小                                                                                                                                                                                                                                                                                              |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

`GET /api/v1/open_orders`

检索尚未匹配或最近已匹配的未完成订单。

### 请求参数

| 名称               | 类型    | 是否必须 | 描述                                                                                    |
| ---                | ---     | ---      | ---                                                                                     |
| symbol             | string  | No       | 市场符号                                                                                 |
| orderID            | string  | No       | 使用内部订单ID查询                                                                        |
| clOrderID          | string  | No       | 使用自定义订单ID查询。如果提供了`orderID`，`clOrderID`将被忽略。                                   |

### 响应内容

| 名称                         | 类型    | 是否必须 | 描述                                                                                  |
| ---                          | ---     | ---      | ---                                                                                   |
| symbol                       | string  | Yes      | 市场符号                                                                               |
| clOrderID                    | string  | Yes      | 交易员发送的客户标签                                                                   |
| filledSize                   | long    | Yes      | 已成交的交易量                                                                         |
| orderValue                   | double  | Yes      | 名义价值                                                                               |
| pegPriceMin                  | double  | Yes      | 最小挂钩价格                                                                           |
| pegPriceMax                  | double  | Yes      | 最大挂钩价格                                                                           |
| pegPriceDeviation            | double  | Yes      | 偏差百分比。仅适用于Algo订单                                                           |
| cancelDuration               | long    | Yes      | 以毫秒为单位的过期时间。<br/>0: GTC<br/>-1: IOC                                        |
| orderID                      | string  | Yes      | 订单ID                                                                                 |
| orderType                    | integer | Yes      | 订单类型 <br/>76: 限价单<br/>77: 市价单<br/>80: Algo订单                               |
| timeInForce                  | string  | Yes      | 订单有效期                                                                             |
| price                        | double  | Yes      | 订单价格                                                                               |
| side                         | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                               |
| size                         | long    | Yes      | 合同大小中的订单大小                                                                   |
| timestamp                    | long    | Yes      | 订单时间戳                                                                             |
| triggerOrder                 | bool    | Yes      | 指示这是否为触发订单                                                                   |
| triggered                    | bool    | Yes      | 指示此订单是否已被触发                                                                 |
| triggerUseLastPrice          | bool    | Yes      | 指示此触发订单是否使用最后价格                                                         |
| triggerPrice                 | double  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                              |
| triggerOriginalPrice         | double  | Yes      | 原始触发价格                                                                           |
| triggerOrderType             | string  | Yes      | 触发订单类型 <br/>1001: 触发止损 <br/>1002: 触发获利                                   |
| triggerTrailingStopDeviation | double  | Yes      | 保留属性                                                                               |
| triggerStopPrice             | double  | Yes      | 保留属性                                                                               |
| trailValue                   | double  | Yes      | 保留属性                                                                               |
| reduceOnly                   | bool    | Yes      | 指示此订单是否仅为减少                                                                 |
| avgFilledPrice               | double  | Yes      | 平均成交价格。返回部分交易订单的平均成交价格                                           |
| averageFillPrice             | double  | Yes      | 平均成交价                                                                             |
| stealth                      | double  | Yes      | 订单的隐身值                                                                           |
| orderState                   | string  | Yes      | `STATUS_ACTIVE`, `STATUS_INACTIVE`                                                     |
| takeProfitOrder    | TakeProfitOrder对象  | No | 止盈订单信息 |
| stopLossOrder      | StopLossOrder对象    | No | 止损订单信息 |
| closeOrder         | bool                | Yes | 是否为关闭此持仓的订单 |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |
| contractSize                 | double   | Yes      | 订单合约规模                                                              |

## 查询成交记录

> 请求

```
/api/v1/trade_history?symbol=BTC-PERP
```

> 响应

```json
[
  {
    "base": "string",
    "clOrderID": "string",
    "feeAmount": 0,
    "feeCurrency": "string",
    "filledPrice": 0,
    "filledSize": 0,
    "averageFillPrice": 0,
    "orderId": "string",
    "orderType": 0,
    "price": 0,
    "quote": "string",
    "realizedPnl": 0,
    "serialId": 0,
    "side": "string",
    "size": 0,
    "symbol": "string",
    "timestamp": 0,
    "total": 0,
    "tradeId": "string",
    "triggerPrice": 0,
    "triggerType": 0,
    "username": "string",
    "positionId": null,
    "wallet": "string",
    "tradeId": "string",
    "orderId": "string",
    "contractSize": "number"
  }
]
```

`GET /api/v1/trade_history`

获取用户的交易历史

### 请求参数

| 名称              | 类型    | 是否必须 | 描述                                                                                               |
| ---               | ---     | ---      | ---                                                                                                |
| symbol            | string  | No       | 市场符号                                                                                            |
| startTime         | long    | No       | 开始时间 (例如：1624987283000)                                                                      |
| endTime           | long    | No       | 结束时间 (例如：1624987283000)                                                                      |
| beforeSerialId    | string  | No       | 获取指定序列ID之前的记录的条件。用于分页                                                             |
| afterSerialId     | string  | No       | 获取指定序列ID之后的记录的条件。用于分页                                                             |
| count             | long    | No       | 返回的记录数量                                                                                      |
| includeOld        | boolean | No       | 检索过去7天的交易历史记录                                                                            |
| orderID           | string  | No       | 通过订单ID查询交易历史                |
| clOrderID         | string  | No       | 通过自定义订单ID查询交易历史                                                                         |

### 响应内容

| 名称             | 类型    | 是否必须 | 描述                                                                                                                                                                                 |
| ---              | ---     | ---      | ---                                                                                                                                                                                  |
| symbol           | string  | Yes      | 市场符号                                                                                                                                                                            |
| side             | string  | Yes      | 交易方向。可取值为: [`BUY`, `SELL`]                                                                                                                                                  |
| price            | double  | Yes      | 成交价格                                                                                                                                                                             |
| size             | long    | Yes      | 原始订单数量                                                                                                                                                                             |
| serialId         | long    | Yes      | 序列号，连续的序列号                                                                                                                                                                 |
| tradeId          | string  | Yes      | 交易标识符                                                                                                                                                                          |
| timestamp        | long    | Yes      | 成交时间戳                                                                                                                                                                          |
| base             | string  | Yes      | 基础货币                                                                                                                                                                            |
| quote            | string  | Yes      | 报价货币                                                                                                                                                                            |
| wallet           | string  | Yes      | 钱包名称<br/>`CROSS@`: 跨钱包<br/>`ISOLATED@market`: Market指的是当前的符号，后面跟`-USD`。例如，BTC-PERP的独立钱包为`ISOLATED@BTC-PERP-USDT`                                               |
| clOrderID        | string  | Yes      | 自定义订单ID                                                                                                                                                                         |
| orderId          | string  | Yes      | 订单ID                                                                                                                                                                              |
| username         | string  | Yes      | btse 用户名                                                                                                                                                                          |
| triggerType      | long    | Yes      | 触发类型<br/>1001: 止损<br/>1002: 获利                                                                                                                                              |
| feeAmount        | long    | Yes      | 费用金额                                                                                                                                                                            |
| feeCurrency      | long    | Yes      | 费用货币                                                                                                                                                                            |
| filledPrice      | double  | Yes      | 平均成交价格                                                                                                                                                                        |
| averageFillPrice | double  | Yes      | 平均成交价格                                                                                                                                                                        |
| triggerPrice     | double  | Yes      | 触发价格                                                                                                                                                                            |
| filledSize       | long    | Yes      | 成交大小                                                                                                                                                                            |
| orderType        | integer | Yes      | 订单类型                                                                                                                                                                            |
| realizedPnL      | double  | Yes      | 现货中未使用                                                                                                                                                                        |
| total            | long    | Yes      | 现货中未使用                                                                                                                                                                        |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |
| contractSize     | double  | Yes      | 交易合约规模                                                                                                                                                           |


## 查询持仓

> 请求

```
/api/v1/positions?symbol=BTC-PERP
```

> 响应

```json
[
  {
    "marginType": 0,
    "entryPrice": 0,
    "markPrice": 29286.4,
    "symbol": "BTC-PERP",
    "side": "BUY",
    "orderValue": 441.8492,
    "settleWithAsset": "BTC",
    "unrealizedProfitLoss": -0.23538014,
    "totalMaintenanceMargin": 2.366912551,
    "size": 62,
    "liquidationPrice": 0,
    "isolatedLeverage": 25,
    "adlScoreBucket": 2,
    "liquidationInProgress": false,
    "timestamp": 1576661434072,
    "currentLeverage": 0,
    "takeProfitOrder": {
        "orderId": "ea1ab233-c79a-4503-a475-f8633ecc9d79",
        "side": "SELL",
        "triggerPrice": 31000.0,
        "triggerUseLastPrice": false
    },
    "stopLossOrder": {
        "orderId": "48523190-77b9-44ea-bee0-d67a428a51b8",
        "side": "SELL",
        "triggerPrice": 27000.0,
        "triggerUseLastPrice": true
    },
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT"
  },{
     "marginType": 91,
     "entryPrice": 1631.106666667,
     "markPrice": 1630.398947255,
     "symbol": "ETH-PERP",
     "side": "BUY",
     "orderValue": 48.9119684176,
     "settleWithAsset": "USDT",
     "unrealizedProfitLoss": -0.02123158,
     "totalMaintenanceMargin": 0.254871114,
     "size": 3,
     "liquidationPrice": 0,
     "isolatedLeverage": 0,
     "adlScoreBucket": 2,
     "liquidationInProgress": false,
     "timestamp": 0,
     "takeProfitOrder": null,
     "stopLossOrder": null,
     "positionMode": "HEDGE",
     "positionDirection": "LONG",
     "positionId": "ETH-PERP-USDT|LONG",
     "currentLeverage": 0.0340349245,
     "takeProfitOrder": null,
     "stopLossOrder": null
     }
]
```

`GET /api/v1/positions`

查询用户当前的仓位。当未指定交易对时，将返回所有市场的仓位。

### 请求参数

| 名称               | 类型    | 是否必须 | 描述                                                                                     |
| ---                | ---     | ---      | ---                                                                                      |
| symbol             | string  | No       | 市场符号                                                                                  |

### 响应内容

| 名称                   | 类型    | 是否必须 | 描述                                                                                              |
| ---                    |---------| ---      | ---                                                                                               |
| symbol                 | string  | Yes      | 市场符号                                                                                           |
| side                   | string  | Yes      | 仓位方向。可取值为: [`Buy`, `SELL`]                                                                 |
| size                   | long    | Yes      | 仓位大小                                                                                           |
| entryPrice             | double  | Yes      | 入场价格                                                                                           |
| markPrice              | double  | Yes      | 标记价格                                                                                           |
| marginType             | long    | Yes      | 保证金类型。值如下<br/>91: CROSS钱包<br/>92: 独立钱包                                                 |
| orderValue             | double  | Yes      | 名义价值                                                                                           |
| settleWithAsset        | string  | Yes      | 结算货币                                                                                           |
| totalMaintenanceMargin | double  | Yes      | 维持保证金                                                                                         |
| unrealizedProfitLoss   | double  | Yes      | 未实现的利润和损失                                                                                  |
| liquidationPrice       | double  | Yes      | 清算价格                                                                                           |
| isolatedLeverage       | double  | Yes      | 独立杠杆值                                                                                         |
| adlScoreBucket         | double  | Yes      | ADL得分概率                                                                                        |
| liquidationInProgress  | boolean | Yes      | 指示是否正在进行清算                                                                               |
| currentLeverage        | double  | Yes      | 当前杠杆                                                                                           |
| timestamp              | long    | Yes      | 查询仓位时的时间戳                                                                                 |
| takeProfitOrder  | TakeProfitOrder对象 | No | 止盈订单信息 |
| stopLossOrder    | StopLossOrder对象   | No | 止损订单信息 |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 平仓仓位

> 请求

```json
{
  "price": 0,
  "symbol": "BTC-PERP",
  "type": "MARKET"
}
```
> 请求（用于双向持仓订单）

```json
{
  "price": 0,
  "symbol": "BTC-PERP",
  "type": "MARKET",
  "positionId": "BTC-PERP-USDT|LONG"
}
```
> Response

```json
[
  {
    "status": 4,
    "symbol": "BTC-PERP",
    "orderType": 76,
    "price": 24010.0,
    "side": "SELL",
    "size": 1,
    "orderID": "93cf814a-595e-4b20-bba9-5c5340ca947d",
    "timestamp": 1660710188450,
    "triggerPrice": 0.0,
    "trigger": false,
    "deviation": 100.0,
    "stealth": 100.0,
    "message": "",
    "avgFillPrice": 24010.0,
    "fillSize": 1.0,
    "clOrderID": "",
    "originalSize": 1.0,
    "postOnly": false,
    "remainingSize": 0.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": null,
    "time_in_force": "GTC"
  }
]
```

`POST /api/v1/order/close_position`

平仓用户在特定市场上指定的仓位。如果指定类型为LIMIT，则价格是必须的。当类型为MARKET时，以市场价格平仓仓位。

### 请求参数

| 名称               | 类型    | 是否必须 | 描述                                                                                                                 |
|--------------------| ---     | ---      | --------------------------------------------------------------------------------------------------------------------|
| symbol             | string  | Yes      | 市场符号                                                                                                             |
| type               | string  | Yes      | 平仓类型，其值为：<br/>LIMIT: 以`price`价格平仓<br/>MARKET: 以市价平仓                                                 |
| price              | double  | No       | 平仓价格。当类型为`LIMIT`时，此字段为必填                                                                           |
| postOnly           | boolean | No       | 布尔值，表示这是否只做Maker(Post only) 订单，交易者将支付Maker手续费                                           |
| positionId         | string  | No       | 想要平仓的仓位ID。在非单向持仓时为必填项                        |

### 响应内容

| 名称           | 类型    | 是否必须 | 描述                                                                                                                                                                                                                                                                          |
| ---            | ---     | ---      | ---                                                                                                                                                                                                                                                                           |
| symbol         | string  | Yes      | 市场符号                                                                                                                                                                                                                                                                     |
| clOrderID      | string  | Yes      | 交易员发送的客户标签                                                                                                                                                                                                                                                        |
| fillSize       | string  | Yes      | 已成交的交易量                                                                                                                                                                                                                                                               |
| orderID        | string  | Yes      | 订单ID                                                                                                                                                                                                                                                                       |
| orderType      | integer | Yes      | 订单类型 <br/>76: 限价单<br/>77: 市价单<br/>80: Algo订单                                                                                                                                                                                                                     |
| postOnly       | boolean | Yes      | 表示订单是否仅为只做Maker(Post only) 订单                                                                                                                                                                                                                                                     |
| price          | double  | Yes      | 订单价格                                                                                                                                                                                                                                                                     |
| side           | string  | Yes      | 订单方向<br/>BUY 或 SELL                                                                                                                                                                                                                                                    |
| size           | long    | Yes      | 已取消的大小                                                                                                                                                                                                                                                                |
| status         | long    | Yes      | 订单状态<br/>2: 已插入订单<br/>3: 已交易订单<br/>4: 订单已全部交易<br/>5: 订单部分交易<br/>6: 已取消订单<br/>7: 已退款订单<br/>9: 触发器已插入<br>10: 触发器已激活<br/>15: 订单被拒绝<br/>16: 找不到订单<br/>17: 请求失败                                                   |
| time_in_force  | string  | Yes      | 订单有效性                                                                                                                                                                                                                                                                  |
| timestamp      | long    | Yes      | 订单时间戳                                                                                                                                                                                                                                                                  |
| trigger        | string  | Yes      | 指示订单是否为触发订单                                                                                                                                                                                                                                                      |
| triggerPrice   | string  | Yes      | 订单触发价格，如果订单不是触发订单则返回0                                                                                                                                                                                                                                   |
| avgFillPrice   | string  | Yes      | 平均成交价格。返回部分交易订单的平均成交价格                                                                                                                                                                                                                                |
| message        | string  | Yes      | 交易消息                                                                                                                                                                                                                                                                    |
| stealth        | double  | Yes      | 订单的隐身值                                                                                                                                                                                                                                                                |
| deviation      | string  | Yes      | 订单的偏差值                                                                                                                                                                                                                                                                |
| remainingSize  | double  | Yes      | 剩余待交易的大小                                                                                                                                                                                                                                                           |
| originalSize   | double  | Yes      | 原始订单大小                                                                                                                                                                                                                                                               |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 设置杠杆

> 请求

```json
{
  "symbol": "BTC-PERP",
  "leverage": 0,
  "marginMode": "CROSS"
}
```

> 请求 (当双向持仓时)

```json
{
    "symbol": "BTC-PERP",
    "leverage": 0,
    "positionMode": "HEDGE",
    "marginMode": "CROSS"
}
```

> 响应

```json
{
  "symbol": "BTC-PERP",
  "timestamp": 1660711246942,
  "status": 20,
  "type": 93,
  "message": ""
}
```

`POST /api/v1/leverage`

更改指定市场的杠杆值

### 请求参数

| 名称               | 类型    | 是否必须 | 描述                                                                                     |
| ---                | ---     | ---      | ---                                                                                      |
| symbol             | string  | Yes      | 市场符号                                                                                 |
| leverage           | long    | Yes      | 杠杆值                                                                                   |
| positionMode       | string  | no       | 单向持仓`ONE_WAY`（默认）或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`, 在非单向持仓时为必填项                                                            |
| positionId         | string  | No       | 在逐仓保证金模式想要设置的仓位ID。                       |
| marginMode       | string  | no       | `CROSS` 或 `ISOLATED`(默认)                                                            |

### 响应内容

| 名称       | 类型    | 是否必须 | 描述                                                                                                                                                          |
| ---        | ---     | ---      | ---                                                                                                                                                           |
| symbol     | string  | Yes      | 市场符号                                                                                                                                                      |
| status     | long    | Yes      | 请求的状态。可取值为：<br/>8: 余额不足<br/>13: 无效的杠杆<br/>20: 成功<br/>64: 正在进行的清算                                                                                             |
| type       | double  | Yes      | 值将为93，表示类型为`杠杆`                                                                                                                                    |
| timestamp  | long    | Yes      | 设置杠杆的时间戳                                                                                                                                               |
| message    | long    | Yes      | 消息                                                                                                                                                          |

## 获取杠杆

> 响应

```json
{
  "symbol": "BTC-PERP",
  "leverage": 100.0,
  "marginMode": "CROSS"
}
```

`Get /api/v1/leverage`

获取指定市场的杠杆值

### 请求参数

| 名称     | 类型    | 是否必须 | 描述       |
| ---      | ---     | ---      | ---        |
| symbol   | string  | Yes      | 市场符号   |

### 响应内容

| 名称      | 类型    | 是否必须 | 描述                                                                                                       |
| ---       | ---     | ---      | ---                                                                                                        |
| symbol    | string  | Yes      | 市场符号                                                                                                   |
| leverage  | double  | Yes      | 当前市场的杠杆值，返回 0 表示杠杆是最大的全仓杠杆。                                       |
| marginMode| string  | Yes      | 当前保证金模式                                                                                  |

## 绑定止盈/止损
> 请求

```json
{
    "symbol": "BTC-PERP",
    "takeProfitPrice": 31000,
    "takeProfitTrigger": "markPrice",
    "stopLossPrice": 22000,
    "stopLossTrigger": "lastPrice"
}
```

> 响应

```json
[
    {
        "status": 9,
        "symbol": "BTC-PERP",
        "orderType": 77,
        "price": 0.0,
        "side": "SELL",
        "size": 100,
        "orderID": "4820b20a-e41b-4273-b3ad-4b19920aeeb5",
        "timestamp": 1691974463934,
        "triggerPrice": 31000.0,
        "trigger": true,
        "deviation": 100.0,
        "stealth": 100.0,
        "message": "",
        "avgFillPrice": 0.0,
        "fillSize": 0.0,
        "clOrderID": "",
        "originalSize": 100.0,
        "postOnly": false,
        "remainingSize": 100.0,
        "orderDetailType": null,
        "positionMode": "ONE_WAY",
        "positionDirection": null,
        "positionId": "BTC-PERP-USDT",
        "time_in_force": "GTC"
    }
]
```

`POST /api/v1/order/bind/tpsl`

绑定止盈/止损与已有持仓

### 请求参数

| 名称               | 类型    | 是否必需 | 描述 |
| ---                | ---     | ---      | --- |
| symbol             | string  | Yes       | 市场交易对 |
| side               | string  | Yes       | `BUY` 或 `SELL` 在双向持仓时为必填项, 在双向持仓时，买方`BUY`綁定至空头仓位，卖方`SELL`则綁定至多头仓位 |
| takeProfitPrice    | double  | No       | 创建带有止盈订单时强制执行。指示触发价格。在使用此API时，必须至少设置`takeProfitPrice`或`stopLossPrice`。 |
| takeProfitTrigger  | string  | No       | 用于创建带有止盈订单的选项。有效选项：`标记价格`（默认）或`最新价格` |
| stopLossPrice      | double  | No       | 创建带有止损订单时强制执行。指示触发价格 |
| stopLossTrigger     | string  | No       | 用于创建带有止损订单的选项。有效选项：`标记价格`（默认）或`最新价格` |
| positionMode       | string  | no       | 单向持仓`ONE_WAY`（默认）或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`, 在非单向持仓时为必填项                                                            |
| positionId         | string  | No       | 想要设置的仓位ID。 在逐仓保证金模式为必填项                      |

### 响应内容

| 名称          | 类型    | 是否必需 | 描述 |
| ---           | ---     | ---      | --- |
| symbol        | string  | Yes       | 市场交易对 |
| clOrderID     | string  | Yes       | 交易员发送的客户标签 |
| fillSize      | number  | Yes       | 成交的交易量 |
| orderID       | string  | Yes       | 订单ID |
| orderType     | string  | Yes       | 订单类型 <br/>76: 限价订单<br/>77: 市价订单<br/>80: 算法订单 |
| postOnly      | boolean  | Yes       | 指示订单是否为只做Maker(Post only) 订单  |
| price         | double  | Yes       | 订单价格 |
| side          | string  | Yes       | 订单方向<br/>BUY 或 SELL |
| size          | long  | Yes       | 以"合约大小"为单位的订单大小（即使在风险限制调整后，此值也保持不变）|
| status        | long  | Yes       | 订单状态<br/>2: 订单已插入<br/>3: 订单已成交<br/>4: 订单已完全成交<br/>5: 订单部分成交<br/>6: 订单已取消<br/>7: 订单已退款<br/>9: 触发已插入<br>10: 触发已激活<br>15: 订单已拒绝<br>16: 未找到订单<br>17: 请求失败 |
| time_in_force | string  | Yes       | 订单有效期 |
| timestamp     | long  | Yes       | 订单时间戳  |
| trigger       | boolean  | Yes       | 指示订单是否为触发订单 |
| triggerPrice  | double  | Yes       | 订单触发价格，如果订单不是触发订单，则返回0 |
| avgFillPrice  | double  | Yes       | 平均成交价格。对于部分成交订单，返回平均成交价格 |
| message       | string  | Yes       | 交易消息  |
| stealth       | string  | Yes       | 仅适用于算法订单 |
| deviation     | double  | Yes       | 仅适用于算法订单 |
| remainingSize | double  | Yes       | 剩余待成交的大小 |
| originalSize  | double  | Yes       | 原始订单大小    |
| positionMode      | string  | Yes      | 仓位模式<br/> 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED`                                                                                                                                                                                                                                                                                  |
| positionDirection | string  | Yes      | 仓位方向<br/>  多头仓位`LONG` 或 空头仓位`SHORT`                                                                                                                                                                                                                                                                             |
| positionId        | string  | Yes      | 当前订单属于的仓位ID。                                                                                                                                                                                                                                                                             |

## 查询仓位模式

> 响应

```json
[
    {
        "symbol": "ETH-PERP",
        "positionMode": "HEDGE"
    },
    {
        "symbol": "BTC-PERP",
        "positionMode": "ONE_WAY"
    }
]
```

`GET /api/v1/position_mode`

查询用户的仓位模式

**请求参数**

| Name               | Type    | Required | Description |
| ---                | ---     | ---      | ------------|
| symbol             | string  | No       | 市场交易对    |

**响应内容**

| Name         | Type   | Required | Description                       |
| ---          | ---    | ---      | --- ------------------------------|
| symbol       | string | Yes      | 市场交易对                         |
| positionMode | string | Yes      | 单向持仓`ONE_WAY` 或  双向持仓`HEDGE` 或 逐仓保证金模式`ISOLATED` |

## 更改仓位模式

> 响应

```json
{
  "symbol": "BTC-PERP",
  "positionMode": "HEDGE"
}
```