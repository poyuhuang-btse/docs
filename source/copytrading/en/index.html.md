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

# Change Log

## Version 1.0.0 (28th June 2025)

* Initial release of the copy trading api

# Overview

## Generating API Key

You will need to create an API key on the BTSE platform before you can use authenticated APIs. To create API keys, you can follow the steps below:

* Login with your username / email and password into the BTSE website
* Click on “Account” on the top right hand corner
* Select the API tab
* Click on “New API” button to create an API key and passphrase. (Note: the passphrase will only appear once)
* Use your API key and passphrase to construct a signature.

## Endpoints

* Production
  * HTTP
     * `https://api.btse.com/copytrading`
  * Websocket
     * `wss://ws.btse.com/ws/futures`
* Testnet
  * HTTP
     * `https://testapi.btse.io/copytrading`
  * Websocket
     * `wss://testws.btse.io/ws/futures`

## Authentication

* API Key (request-api)
  * Parameter Name: `request-api`, in: header. API key is obtained from BTSE platform as a string

* API Key (request-nonce)
  * Parameter Name: `request-nonce`, in: header. Representation of current timestamp in long format

* API Key (request-sign)
  * Parameter Name: `request-sign`, in: header. A composite signature produced based on the following algorithm: Signature=HMAC.Sha384 (secretkey, (urlpath + request-nonce + bodyStr)) (note: bodyStr = '' when no data):

### Example: Place an order

> **HMAC SHA384 Signature**

```shell
$ echo -n "/api/v1/order1624985375123{\"postOnly\":false,\"price\":8500.0,\"reduceOnly\":false,\"side\":\"BUY\",\"size\":1,\"stopPrice\":0.0,\"symbol\":\"BTC-PERP\",\"time_in_force\":\"GTC\",\"trailValue\":0.0,\"triggerPrice\":0.0,\"txType\":\"LIMIT\",\"type\":\"LIMIT\"}" | openssl dgst -sha384 -hmac "848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx"
(stdin)= 943adfce43b609a28506274976b96e08cf4bdc4ea53ca0b4cac0eb2cf0773a7d0807efc0aeab779d47fadcd9a60eea13
```

* Endpoint to place an order is `https://api.btse.com/copytrading/api/v1/order`
* Assume we have the values as follows:
  * request-nonce: `1624985375123`
  * request-api: `4e9536c79f0fdd72bf04f2430982d3f61d9d76c996f0175bbba470d69d59816x`
  * secret: `848db84ac252b6726e5f6e7a711d9c96d9fd77d020151b45839a5b59c37203bx`
  * Path: `/api/v1/order`
  * Body: `{"postOnly":false,"price":8500.0,"reduceOnly":false,"side":"BUY","size":1,"stopPrice":0.0,"symbol":"BTC-PERP","time_in_force":"GTC","trailValue":0.0,"triggerPrice":0.0,"txType":"LIMIT","type":"LIMIT"}`
  * Encrypted Text: `/api/v1/order1624985375123{"postOnly":false,"price":8500.0,"reduceOnly":false,"side":"BUY","size":1,"stopPrice":0.0,"symbol":"BTC-PERP","time_in_force":"GTC","trailValue":0.0,"triggerPrice":0.0,"txType":"LIMIT","type":"LIMIT"}`
* Generated signature will be:
  * request-sign: `943adfce43b609a28506274976b96e08cf4bdc4ea53ca0b4cac0eb2cf0773a7d0807efc0aeab779d47fadcd9a60eea13`


## Rate Limits

* The following rate limits are enforced:

Rate limits for BTSE is as follows:

**Query**

* Per API: `15 requests/second`
* Per User: `30 requests/second`

**Orders**

* Per API: `75 requests/second`
* Per User: `75 requests/second`

### Mechanism Description

Our system implements a tiered blocking mechanism with three distinct durations: **1 second**, **5 minutes**, and **15 minutes**. The duration of the block begins calculation from the moment the first block is imposed.
Additionally, the calculation duration will be reset if the IP address or user does not exceed the rate limit within a span of 1 hour or the 15 mins blocking duration is ended.

A `Retry-After` header is included with a 429 response and will give the unlocked timestamp.

#### Rate limit tiers

* 1 sec
* 5 min
* 15 min

## API Status Codes

Each API will return one of the following HTTP status:

* 200 - API request was successful, refer to the specific API response for expected payload
* 400 - Bad Request. Server will not process this request. This is usually due to invalid parameters sent in request
* 401 - Unauthorized request. Server will not process this request as it does not have valid authentication credentials
* 403 - Forbidden request. Credentials were provided but they were insufficient to perform the request
* 404 - Not found. Indicates that the server understood the request but could not find a correct representation for the target resource
* 405 - Method not allowed. Indicates that the request method is not known to the requested server
* 408 - Request timeout. Indicates that the server did not complete the request. BTSE API timeouts are set at 30secs
* 429 - Too many requests. Indicates that the client has exceeded the rates limits set by the server. Refer to Rate Limits for more details
* 451 - Unavailable For Legal Reasons. Indicates that the client has been banned because abnormal behavior
* 500 - Internal server error. Indicates that the server encountered an unexpected condition resulting in not being able to fulfill the request

## API Enum

When connecting up the BTSE API, you will come across number codes that represents different states or status types in BTSE. The following section provides a list of codes that you are expecting to see.

* 1: MARKET_UNAVAILABLE = Futures market is unavailable
* 2: ORDER_INSERTED = Order is inserted successfully
* 4: ORDER_FULLY_TRANSACTED = Order is fully transacted
* 5: ORDER_PARTIALLY_TRANSACTED = Order is partially transacted
* 6: ORDER_CANCELLED = Order is cancelled successfully
* 7: ORDER_REFUNDED = Order is refunded
* 8: INSUFFICIENT_BALANCE = Insufficient balance in account
* 9: TRIGGER_INSERTED = Trigger Order is inserted successfully
* 10: TRIGGER_ACTIVATED = Trigger Order is activated successfully
* 11: ERROR_INVALID_CURRENCY
* 12: ERROR_UPDATE_RISK_LIMIT = Error in updating risk limit
* 13: ERROR_INVALID_LEVERAGE
* 15: ORDER_REJECTED = Order is rejected
* 16: ORDER_NOTFOUND = Order is not found with the order ID or clOrderID provided
* 17: REQUEST_FAILED = Failed to complete the request, please check order status
* 20: SUCCESS = Action succeeded.
* 21: FREEZE_SUCCESSFUL
* 27: TRANSFER_SUCCESSFUL = Transfer funds between futures and spot is successful
* 28: TRANSFER_UNSUCCESSFUL = Transfer funds between spot and futures is unsuccessful
* 29: QUERY_GET_ORDERS
* 31: QUERY_GET_POSITIONS
* 33: QUERY_GET_ALL_POSITIONS_ORDERS
* 34: QUERY_WALLET
* 36: QUERY_FUTURES_MARGIN
* 41: ERROR_INVALID_RISK_LIMIT = Invalid risk limit was specified
* 51: QUERY_GET_ORDERS_WITH_LIMIT
* 64: STATUS_LIQUIDATION = Account is undergoing liquidation
* 65: STATUS_ACITVE = Order is active
* 66: MODE_BUY
* 76: ORDER_TYPE_LIMIT = Limit order
* 77: ORDER_TYPE_MARKET = Market order
* 80: ORDER_TYPE_PEG = Peg/Algo order
* 81: ORDER_TYPE_OTC = Otc order
* 83: MODE_SELL
* 85: STATUS_PROCESSING = Order is inactive
* 88: STATUS_INACTIVE = Order is inactive
* 101: FUTURES_ORDER_PRICE_OUTSIDE_LIQUIDATION_PRICE = Futures order is outside of liquidation price
* 110: FUTURES_FUNDING
* 123: AMEND_ORDER = Order amended
* 124: UNFREEZE_SUCCESSFUL
* 129: FUTURES_CONFIG_MODE_CHANGE
* 131: FUTURES_STATUS_PROCESSING_LEVERAGE
* 132: FUTURES_STATUS_PROCESSING_RISK_LIMIT
* 133: FUTURES_POSITION_MODE_INVALID
* 134: POSITION_MODE_UNCHANGEABLE
* 138: POSITION_MODE_CHANGE_PROCESSING
* 300: ERROR_MAX_ORDER_SIZE_EXCEEDED
* 301: ERROR_INVALID_ORDER_SIZE
* 302: ERROR_INVALID_ORDER_PRICE
* 303: ERROR_RATE_LIMITS_EXCEEDED
* 304: ERROR_MAX_OPEN_ORDER_EXCEEDED
* 305: ERROR_ORDER_PRICE_OUT_OF_PRICE_PROTECTION_RANGE
* 1003: ORDER_LIQUIDATION = Order is undergoing liquidation
* 1004: ORDER_ADL = Order is undergoing ADL
* 30410: BLOCK_TRADE_COMPLETE_SUCCESS

## Spam Orders

Spam orders are large number of small order sizes that is placed. In order to ensure that the platform and user's interests are protected from malicious players, we will apply the following for users placing small sized orders.

[Spam Order Detection Mechanism : BTSE Support](https://support.btse.com/en/support/solutions/articles/43000720904-spam-order-detection-mechanism)

* Orders with a notional value below 5 USDT will be marked as a spam order and will automatically become hidden orders.
* Orders marked as spam always pay the taker fee.
* Post-Only API orders marked as spam will be rejected instead of being hidden.
* Too many spam orders may be grounds to temporarily ban an account from trading.
* API accounts placing >= 4 resting orders, with total size less than 20 USDT are at risk of being marked as a spam account.
* Accounts marked as spam may have limitations placed on the account, including order rate limits, position limits, or have API functions disabled. For questions regarding the new spam order mechanism, please email mm@btse.com.

# Trade Endpoints

## Create New Order

> Request (create `MARKET` order)

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "side": "BUY",
  "type": "MARKET"
}
```
> Request (create `LIMIT` order)

```json
{
  "symbol": "BTC-PERP",
  "size": 1,
  "price": 21000,
  "side": "BUY",
  "type": "LIMIT"
}
```
> Request (create `LIMIT` `TRIGGER` order)

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
> Request (create `LIMIT` `STOP` order)

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
> Request (create `OCO` order)

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

> Request (create `Limit` order with `TP/SL`)

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
> Request (create `Limit` order with `TP` only)

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

> Request (create `Limit` order with `SL` only)

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

> Response (general)

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

> Response (for `OCO` order)

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

Creates a new order. Requires `Trading` permission

### Request Parameters

| Name          | Type    | Required | Description                                                                                                                                                                                                                                                                                                                                                        |
|---------------| ---     | ---      |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol        | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                                                                                      |
| price         | double  | No       | Mandatory unless creating a MARKET order. Order price                                                                                                                                                                                                                                                                                                              |
| size          | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                                                                                            |
| side          | string  | Yes      | 'BUY' or 'SELL'                                                                                                                                                                                                                                                                                                                                                    |
| time_in_force | string  | No       | Time validity of the order<br/>GTC: Good till Cancel<br/>IOC: Immediate or Cancel<br/>FOK: Fill or Kill<br/>HALFMIN: Order valid for 30 seconds<br/>FIVEMIN: Order valid for 5 mins<br/> HOUR: Order valid for an hour<br/>TWELVEHOUR: Order valid for 12 hours<br/>DAY: Order valid for a day<br/>WEEK: Order valid for a week<br/>MONTH: Order valid for a month |
| type          | string  | Yes      | Order type<br/>LIMIT: Limit Orders<br/>MARKET: Market Orders<br/>OCO: One cancel the other                                                                                                                                                                                                                                                                         |
| txType        | string  | No       | Used for Stop orders or trigger orders<br/>STOP: Stop Order, `triggerPrice` is mandatory<br/>TRIGGER: Trigger order, `triggerPrice` is mandatory<br/>LIMIT: Default, used when its not a Stop order nor Trigger order                                                                                                                                              |
| stopPrice     | double  | No       | Mandatory when creating an OCO order. Indicates the stop price                                                                                                                                                                                                                                                                                                     |
| triggerPrice  | double  | No       | Mandatory when creating a Stop, Trigger, OCO order. Indicates the trigger price                                                                                                                                                                                                                                                                                    |
| trailValue    | double  | No       | Trail value                                                                                                                                                                                                                                                                                                                                                        |
| postOnly      | boolean | No       | Boolean to indicate if this is a post only order. For post only orders, traders are charged maker fees                                                                                                                                                                                                                                                             |
| reduceOnly    | boolean | No       | Boolean to indicate if this is a reduce only order, if in hedge mode, it is used to reduce the specified position, ex: sell to reduce long position, buy to reduce short position.                                                                                                                                                                                 |
| clOrderID     | string  | No       | Custom order Id                                                                                                                                                                                                                                                                                                                                                    |
| trigger       | string  | No       | For creating order with txType: `STOP` or `TRIGGER`. Valid options: `markPrice` (default) or `lastPrice`|
| takeProfitPrice  | double  | No       | Mandatory when creating new order with take profit order. Indicates the trigger price     
| takeProfitTrigger       | string  | No       | For creating order with take profit order. Valid options: `markPrice` (default) or `lastPrice`|
| stopLossPrice  | double  | No       | Mandatory when creating new order with stop loss order. Indicates the trigger price       
| stopLossTrigger       | string  | No       | For creating order with stop loss order. Valid options: `markPrice` (default) or `lastPrice`|
| positionMode  | string  | No       | For creating order and wanting to specify the positionMode. Valid options: `ONE_WAY` (default) 

### Response Content

| Name              | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
|-------------------| ---     |----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID         | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize          | number  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID           | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType         | integer  | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly          | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price             | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side              | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size              | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                         |
| status            | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force     | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp         | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger           | boolean | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice      | double  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice      | double  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message           | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth           | double  | Yes      | Only valid for Algo orders                                                                                                                                                                                                                                                                      |
| deviation         | double  | Yes      | Only valid for Algo orders                                                                                                                                                                                                                                                                      |
| remainingSize     | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize      | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes      | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | The current order belongs to the id of position.                                                                                                                                                                                                                                                |

## Create new algo order

> Request

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

> Response

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

Creates a new algo order. Algo order is an order that price will change according to market price. To create an algo order, user will need to enter additional parameters:

* `price`: What is the min price (for a sell order) or maximum price (for a buy order) that a user will be willing to list his order at
* `deviation`: How much should the order price deviate from index price. Value is in percentage and can range from `-10` to `10`
* `stealth`: How many percent of the order is to be displayed on the orderbook.

This API Requires `Trading` permission

### Request Parameters

| Name         | Type   | Required | Description                                                                                                                                                                       |
|--------------| ---    | ---      | ---                                                                                                                                                                               |
| symbol       | string | Yes      | Market symbol                                                                                                                                                                     |
| price        | double | Yes      | Minimum price for a sell order, this is the lowest price that a user is willing to sell at. Maximum price for a buy order, this is the maximum price a user is willing to buy at. |
| size         | long   | Yes      | Order size                                                                                                                                                                        |
| side         | string | Yes      | Order side<br/>BUY or SELL                                                                                                                                                        |
| clOrderID    | string | No       | Custom order Id                                                                                                                                                                   |
| deviation    | double | No       | How much should the order price deviate from index price. Value is in percentage and can range from `-10` to `10`                                                                 |
| stealth      | double | No       | How many percent of the order is to be displayed on the orderbook.                                                                                                                |
| positionMode | string  | No       | For creating order and wanting to specify the positionMode. Valid options: `ONE_WAY` (default) , `HEDGE` , `ISOLATED`                                                                                                                                                                                                                                                         |

### Response Content

| Name              | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
|-------------------| ---     | ---      |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID         | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize          | number  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID           | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType         | integer  | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly          | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price             | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side              | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size              | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                         |
| status            | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force     | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp         | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger           | boolean | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice      | double  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice      | double  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message           | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth           | double  | Yes      | Stealth value of order                                                                                                                                                                                                                                                                          |
| deviation         | double  | Yes      | Deviation value of order                                                                                                                                                                                                                                                                        |
| remainingSize     | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize      | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes  | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | The current order belongs to the id of position.                                                                                                                                                                                                                                                |

## Amend Order

> Request (amend price)

```json
{
  "symbol": "BTC-PERP",
  "orderID": "604c3ebf-d7fa-468d-9ff0-f6ad030221b4",
  "type": "PRICE",
  "value": 22000
}
```

> Request (amend all)

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

> Response

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

Amend the price or size or trigger price of an order. For trigger orders, if the order has already been triggered, the trigger price cannot be further amended. Amend order _does not_ apply to algo orders

### Request Parameters

| Name         | Type    | Required | Description                                                                                                                                                        |
| ---          | ---     | ---      | ---                                                                                                                                                                |
| symbol       | string  | Yes      | Market symbol                                                                                                                                                      |
| orderID      | string  | No       | Internal order ID. Mandatory when `clOrderID` is not provided. If `orderID` is provided, `clOrderID` will be ignored.                                              |
| clOrderID    | string  | No       | Custom order ID. Mandatory when `orderID` is not provided.                                                                                                         |
| type         | string  | Yes      | Type of amendmend<br/>`PRICE`: To amend order price<br/>`SIZE`: To amend order size<br/>`TRIGGERPRICE`: To amend trigger price<br/>`ALL`: to amend multiple fields |
| value        | number  | Yes      | The value to be amended to. Value depends on the type being set.                                                                                                   |
| orderPrice   | number  | No       | For type: `ALL`, order price to be amended                                                                                                                         |
| orderSize    | number  | No       | For type: `ALL`, order size in contract size to be amended                                                                                                         |
| triggerPrice | number  | No       | For type: `ALL`, trigger price to be amended                                                                                                                       |


### Response Content

| Name              | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
|-------------------| ---     |----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID         | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize          | string  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID           | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType         | integer | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly          | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price             | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side              | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size              | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                         |
| status            | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force     | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp         | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger           | string  | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice      | string  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice      | string  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message           | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth           | double  | Yes      | Stealth value of order                                                                                                                                                                                                                                                                          |
| deviation         | string  | Yes      | Deviation value of order                                                                                                                                                                                                                                                                        |
| remainingSize     | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize      | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes      | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | The current order belongs to the id of position.                                                                                                                                                                                                                                                |

## Cancel Order

> Request

```
/api/v1/order?symbol=BTC-PERP&clOrderID=my-order-id
```

> Response

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

Cancels pending orders that has not yet been transacted. The `orderID` is a unique identifier to cancel a particular order. `clOrderID` is a custom ID sent in by the trader. When cancel by `clOrderID`, all orders having the same ID will be cancelled. If `orderID` and `clOrderID` is not sent in, then cancellation will be for all orders in the current market.

### Request Parameters

| Name      | Type   | Required | Description                                                                                                                        |
| ---       | ---    | ---      | ---                                                                                                                                |
| symbol    | string | Yes      | Market symbol                                                                                                                      |
| orderID   | string | No       | Unique identifier for an order. Mandatory when `clOrderID` is not provided. If `orderID` is provided, `clOrderID` will be ignored. |
| clOrderID | string | No       | Client custom order ID. Mandatory when `orderID` is not provided.                                                                  |


### Response Content

| Name              | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
|-------------------| ---     |----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID         | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize          | double  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID           | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType         | integer | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly          | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price             | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side              | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size              | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                         |
| status            | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force     | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp         | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger           | boolean | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice      | double  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice      | double  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message           | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth           | double  | Yes      | Stealth value of order                                                                                                                                                                                                                                                                          |
| deviation         | double  | Yes      | Deviation value of order                                                                                                                                                                                                                                                                        |
| remainingSize     | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize      | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes      | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | The current order belongs to the id of position.                                                                                                                                                                                                                                                |

## Query Open Orders

> Request

```
/api/v1/open_orders?symbol=BTC-PERP
```

> Response

```json
[
  {
    "orderType": 76,
    "price": 21000.0,
    "size": 1,
    "side": "BUY",
    "filledSize": 0,
    "orderValue": 21.0,
    "pegPriceMin": 0.0,
    "pegPriceMax": 0.0,
    "pegPriceDeviation": 1.0,
    "cancelDuration": 0,
    "timestamp": 1660645487032,
    "orderID": "2eb1c6f5-2ab2-4706-ab88-eea6b710a78b",
    "stealth": 1.0,
    "triggerOrder": false,
    "triggered": false,
    "triggerPrice": 0.0,
    "triggerOriginalPrice": 0.0,
    "triggerOrderType": 0,
    "triggerTrailingStopDeviation": 0.0,
    "triggerStopPrice": 0.0,
    "symbol": "BTC-PERP",
    "trailValue": 0.0,
    "clOrderID": "string",
    "reduceOnly": false,
    "orderState": "STATUS_ACTIVE",
    "triggerUseLastPrice": false,
    "avgFilledPrice": 0.0,
    "positionMode": "ONE_WAY",
    "positionDirection": null,
    "positionId": "BTC-PERP-USDT",
    "timeInForce": "GTC",
    "averageFillPrice": 0.0,
    "contractSize": 0.0001,
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
    "closeOrder": false
  }
]
```

`GET /api/v1/open_orders`

Retrieves open orders that have not yet been matched or matched recently.

### Request Parameters

| Name               | Type    | Required | Description                                                                         |
| ---                | ---     | ---      | ---                                                                                 |
| symbol             | string  | No       | Market symbol                                                                       |
| orderID            | string  | No       | Query using internal order ID                                                       |
| clOrderID          | string  | No       | Query using custom order ID. If `orderID` is provided, `clOrderID` will be ignored. |

### Response Content

| Name                         | Type   | Required | Description                                                                            |
| ---                          | ---    | ---      |----------------------------------------------------------------------------------------|
| symbol                       | string | Yes      | Market symbol                                                                          |
| clOrderID                    | string | Yes      | Customer tag sent in by trader                                                         |
| filledSize                   | long   | Yes      | Trade filled size                                                                      |
| orderValue                   | double | Yes      | Notional value                                                                         |
| pegPriceMin                  | double | Yes      | peg price min                                                                          |
| pegPriceMax                  | double | Yes      | peg price max                                                                          |
| pegPriceDeviation            | double | Yes      | Deviation percentage. Only for Algo orders                                             |
| cancelDuration               | long   | Yes      | Expire in milliseconds. <br/>0: GTC<br/>-1: IOC                                        |
| orderID                      | string | Yes      | Order ID                                                                               |
| orderType                    | integer| Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                |
| timeInForce                  | string | Yes      | Order validity                                                                         |
| price                        | double | Yes      | Order price                                                                            |
| side                         | string | Yes      | Order side<br/>BUY or SELL                                                             |
| size                         | long   | Yes      | Order size in contract size                                                            |
| timestamp                    | long   | Yes      | Order timestamp                                                                        |
| triggerOrder                 | bool   | Yes      | Indicate if this is a trigger order                                                    |
| triggered                    | bool   | Yes      | Indicate if this order has been triggered                                              |
| triggerUseLastPrice          | bool   | Yes      | Indicate if this trigger order uses last price                                         |
| triggerPrice                 | double | Yes      | Order trigger price, returns 0 if order is not a trigger order                         |
| triggerOriginalPrice         | double | Yes      | Original trigger price                                                                 |
| triggerOrderType             | string | Yes      | Trigger order type <br/>1001: Trigger stop loss <br/>1002: Trigger take profit         |
| triggerTrailingStopDeviation | double | Yes      | Reserved attribute                                                                     |
| triggerStopPrice             | double | Yes      | Reserved attribute                                                                     |
| trailValue                   | double | Yes      | Reserved attribute                                                                     |
| reduceOnly                   | bool   | Yes      | Indicate if this order is reduce only                                                  |
| avgFilledPrice               | double | Yes      | Average filled price. Returns the average filled price for partially transacted orders |
| averageFillPrice             | double | Yes      | Average fill price                                                                     |
| stealth                      | double | Yes      | Stealth value of order                                                                 |
| orderState                   | string | Yes      | `STATUS_ACTIVE`, `STATUS_INACTIVE`                                                     |
| takeProfitOrder              | TakeProfitOrder object | No | Take profit order info                                                                 |
| stopLossOrder                | StopLossOrder object   | No | Stop loss order info                                                                   |
| closeOrder                   | bool   | Yes      | Whether it is an order to close this position                                          |
| positionMode                 | string   | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                           |
| positionDirection            | string   | Yes      | Position direction                                                                     |
| positionId                   | string   | Yes      | The current order belongs to the id of position.                                       |
| contractSize                 | double   | Yes      | The order contract size                                                              |

## Query Trades Fills

> Request

```
/api/v1/trade_history?symbol=BTC-PERP
```

> Response

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

Retrieves a user's trade history

### Request Parameters

| Name               | Type    | Required | Description                                                                       |
| ---                | ---     | ---      | ---                                                                               |
| symbol             | string  | No       | Market symbol                                                                     |
| startTime          | long    | No       | Starting time (eg. 1624987283000)                                                 |
| endTime            | long    | No       | Ending time (eg. 1624987283000)                                                   |
| beforeSerialId     | string  | No       | Condition to retrieve records before the specified serial Id. Used for pagination |
| afterSerialId      | string  | No       | Condition to retrieve records after the specified serial Id. Used for pagination  |
| count              | long    | No       | Number of records to return                                                       |
| includeOld         | boolean | No       | Retrieve trade  history records past 7 days                                       |
| orderID            | string  | No       | Query trade history by order ID                                            |
| clOrderID          | string  | No       | Query trade history by custom order ID                                            |

### Response Content

| Name             | Type    | Required | Description                                                                                                                                                                             |
|------------------| ---     | ---      |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol           | string  | Yes      | Market symbol                                                                                                                                                                           |
| side             | string  | Yes      | Trade side. Values are: [`BUY`, `SELL`]                                                                                                                                                 |
| price            | double  | Yes      | Transacted price                                                                                                                                                                        |
| size             | long    | Yes      | Original order size                                                                                                                                                                     |
| serialId         | long    | Yes      | Serial Id, running sequence number                                                                                                                                                      |
| tradeId          | string  | Yes      | Trade identifier                                                                                                                                                                        |
| timestamp        | long    | Yes      | Transacted timestamp                                                                                                                                                                    |
| base             | string  | Yes      | Base currency                                                                                                                                                                           |
| quote            | string  | Yes      | Quote currency                                                                                                                                                                          |
| wallet           | string  | Yes      | Wallet name<br/>`CROSS@`: Cross wallet<br/>`ISOLATED@market`: Market refers to the current symbol with `-USDT` appended. Eg. BTC-PERP isolated wallet would be `ISOLATED@BTC-PERP-USDT` |
| clOrderID        | string  | Yes      | Custom order ID                                                                                                                                                                         |
| orderId          | string  | Yes      | Order ID                                                                                                                                                                                |
| username         | string  | Yes      | btse username                                                                                                                                                                           |
| triggerType      | long    | Yes      | Trigger type<br/>1001: Stop Loss<br/>1002: Take Profit                                                                                                                                  |
| feeAmount        | long    | Yes      | Fee amount                                                                                                                                                                              |
| feeCurrency      | long    | Yes      | Fee currency                                                                                                                                                                            |
| filledPrice      | double  | Yes      | Filled price                                                                                                                                                                            |
| averageFillPrice | double  | Yes      | Average filled price                                                                                                                                                                    |
| triggerPrice     | double  | Yes      | Trigger price                                                                                                                                                                           |
| filledSize       | long    | Yes      | Filled size                                                                                                                                                                             |
| orderType        | integer | Yes      | Order Type                                                                                                                                                                              |
| realizedPnL      | double  | Yes      | Not used in Spot                                                                                                                                                                        |
| total            | long    | Yes      | Not used in Spot                                                                                                                                                                        |
| positionId       | string  | Yes      | The current order belongs to the id of position.                                                                                                                                        |
| contractSize     | double  | Yes      | The trade contract size                                                                                                                                                                 |


## Query Position

> Request

```
/api/v1/positions
```

> Response

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

Queries user's current position. When no symbol is specified, positions for all markets will be returned.

### Request Parameters

| Name               | Type    | Required | Description                                                     |
| ---                | ---     | ---      | ---                                                             |
| symbol             | string  | No       | Market symbol                                                   |

### Response Content

| Name                   | Type    | Required | Description                                                                 |
|------------------------|---------|----------|-----------------------------------------------------------------------------|
| symbol                 | string  | Yes      | Market symbol                                                               |
| side                   | string  | Yes      | Position side. Values are: [`Buy`, `SELL`]                                  |
| size                   | long    | Yes      | Position size                                                               |
| entryPrice             | double  | Yes      | Entry price                                                                 |
| markPrice              | double  | Yes      | Mark price                                                                  |
| marginType             | long    | Yes      | Margin Type. Values as follows<br/>91: CROSS wallet<br/>92: Isolated wallet |
| orderValue             | double  | Yes      | Notional value                                                              |
| settleWithAsset        | string  | Yes      | Settlement currency                                                         |
| totalMaintenanceMargin | double  | Yes      | Maintenance margin                                                          |
| unrealizedProfitLoss   | double  | Yes      | Unrealized profit and loss                                                  |
| liquidationPrice       | double  | Yes      | Liquidation Price                                                           |
| isolatedLeverage       | double  | Yes      | Isolated leverage value                                                     |
| adlScoreBucket         | double  | Yes      | ADL Score probability                                                       |
| liquidationInProgress  | boolean | Yes      | Indicator if liquidation is in progress                                     |
| currentLeverage        | double  | Yes      | Current leverage                                                            |
| timestamp              | long    | Yes      | Timestamp when position was queried                                         |
| takeProfitOrder        | TakeProfitOrder object | No | Take profit order info                                                      |
| stopLossOrder          | StopLossOrder object   | No | Stop loss order info                                                        |
| positionMode           | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                |
| positionDirection      | string  | Yes      | Position direction                                                          |
| positionId             | string  | Yes      | Position id                                                                 |


## Close Position

> Request

```json
{
  "price": 0,
  "symbol": "BTC-PERP",
  "type": "MARKET"
}
```
> Request(For hedge mode position)

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

`DELETE /api/v1/close_position`

Closes a user's position for the particular market as specified by symbol. If type is specified as LIMIT, then price is mandatory. When type is MARKET, it closes the position at market price.

### Request Parameters

| Name               | Type    | Required | Description                                                                                             |
|--------------------| ---     | ---      |---------------------------------------------------------------------------------------------------------|
| symbol             | string  | Yes      | Market symbol                                                                                           |
| type               | string  | Yes      | Close position type with values:<br/>LIMIT: Close at `price`<br/>MARKET: Close at market price          |
| price              | double  | No       | Close price. Mandatory when type is `LIMIT`                                                             |
| postOnly           | boolean | No       | Boolean to indicate if this is a post only order. For post only orders, traders are charged maker fees  |
| positionId         | string  | No       | The position ID that you want to close. Mandatory when positionMode is `HEDGE` or `ISOLATED`                          |

### Response Content

| Name              | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
|-------------------| ---     |----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID         | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize          | string  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID           | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType         | integer | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly          | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price             | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side              | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size              | long    | Yes      | Cancelled size                                                                                                                                                                                                                                                                                  |
| status            | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force     | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp         | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger           | string  | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice      | string  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice      | string  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message           | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth           | double  | Yes      | Stealth value of order                                                                                                                                                                                                                                                                          |
| deviation         | string  | Yes      | Deviation value of order                                                                                                                                                                                                                                                                        |
| remainingSize     | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize      | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes      | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | Position id                                                                                                                                                                                                                                                                                     |

## Get Risk Limit

> Request

```
/api/v1/risk_limit?symbol=BTC-PERP
```

> Response

```json
{
    "symbol": "BTC-PERP",
    "riskLimit": 100000
}
```
`GET /api/v1/risk_limit`

Query risk limit for the specified market
### Request Parameters

| Name               | Type    | Required | Description |
| ---                | ---     | ---      | --- |
| symbol             | string  | Yes      | Market symbol  |

### Response Content

| Name      | Type    | Required | Description|
| ---       | ---     | ---      | --- |
| symbol    | string  | Yes      | Market symbol  |
| riskLimit | long    | Yes      | Risk limit value now in position size, but will be changed to USD value along with futures market name change |

## Get Leverage

> Response

```json
{
  "symbol": "BTC-PERP",
  "leverage": 100.0,
  "marginMode": "ISOLATED"
}
```

`Get /api/v1/leverage`

Get leverage value for the specified market

### Request Parameters

| Name               | Type    | Required | Description |
| ---                | ---     | ---      | --- |
| symbol             | string  | Yes      | Market symbol |

### Response Content

| Name      | Type    | Required | Description                                                                                          |
| ---       | ---     | ---      |------------------------------------------------------------------------------------------------------|
| symbol    | string  | Yes      | Market symbol                                                                                        |
| leverage  | double  | Yes      | Current leverage value for the market, return 0 means the leverage is the maximum cross leverage     |
| marginMode| string  | Yes      | Current margin mode                                                                                  |


## Bind TP/SL
> Request

```json
{
    "symbol": "BTC-PERP",
    "takeProfitPrice": 31000,
    "takeProfitTrigger": "markPrice",
    "stopLossPrice": 22000,
    "stopLossTrigger": "lastPrice"
}
```

> Response

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

Bind TP/SL with an existing position

### Request Parameters

| Name               | Type    | Required | Description
| ---                | ---     | ---      | --- 
| symbol             | string  | yes       | Market symbol
| side               | string  | yes       | "BUY" or "SELL" Mandatory when positionMode is `HEDGE`, in hedge mode, it is used to clsoe the specified position, ex: sell to close long position, buy to close short position
| takeProfitPrice    | double  | No        | Mandatory when creating new order with take profit order. Indicates the trigger price. Must set takeProfitPrice or stopLossPrice at least when using this API. |
| takeProfitTrigger  | string  | No        | For creating order with take profit order. Valid options: `markPrice` (default) or `lastPrice` |
| stopLossPrice      | double  | No        | Mandatory when creating new order with stop loss order. Indicates the trigger price        |
| stopLossTrigger     | string | No       | For creating order with stop loss order. Valid options: `markPrice` (default) or `lastPrice`|
| positionMode       | string  | no       | ONE_WAY(default) or HEDGE or ISOLATED. Mandatory when positionMode is `HEDGE` or `ISOLATED` |
| positionId         | string  | no       | The position ID that you want to bind. Mandatory when positionMode is `ISOLATED` |

### Response Content

| Name          | Type    | Required | Description                                                                                                                                                                                                                                                                                     |
| ---           | ---     | ---      |-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol        | string  | Yes      | Market symbol                                                                                                                                                                                                                                                                                   |
| clOrderID     | string  | Yes      | Customer tag sent in by trader                                                                                                                                                                                                                                                                  |
| fillSize      | number  | Yes      | Trade filled size                                                                                                                                                                                                                                                                               |
| orderID       | string  | Yes      | Order ID                                                                                                                                                                                                                                                                                        |
| orderType     | string  | Yes      | Order type <br/>76: Limit Order<br/>77: Market order<br/>80: Algo order                                                                                                                                                                                                                         |
| postOnly      | boolean | Yes      | Indicates if order is a post only order                                                                                                                                                                                                                                                         |
| price         | double  | Yes      | Order price                                                                                                                                                                                                                                                                                     |
| side          | string  | Yes      | Order side<br/>BUY or SELL                                                                                                                                                                                                                                                                      |
| size          | long    | Yes      | Order size in `contract size` (this remains unchanged even after risk limit adjustment)                                                                                                                                                                                                         |
| status        | long    | Yes      | Order status<br/> 2: Order Inserted<br/>3: Order Transacted<br/>4: Order Fully Transacted<br/>5: Order Partially Transacted<br/>6: Order Cancelled<br/>7: Order Refunded<br/>9: Trigger Inserted<br>10: Trigger Activated<br/>15: Order Rejected<br/>16: Order Not Found<br/>17: Request failed |
| time_in_force | string  | Yes      | Order validity                                                                                                                                                                                                                                                                                  |
| timestamp     | long    | Yes      | Order timestamp                                                                                                                                                                                                                                                                                 |
| trigger       | boolean | Yes      | Indicator if order is a trigger order                                                                                                                                                                                                                                                           |
| triggerPrice  | double  | Yes      | Order trigger price, returns 0 if order is not a trigger order                                                                                                                                                                                                                                  |
| avgFillPrice  | double  | Yes      | Average filled price. Returns the average filled price for partially transacted orders                                                                                                                                                                                                          |
| message       | string  | Yes      | Trade messages                                                                                                                                                                                                                                                                                  |
| stealth       | string  | Yes      | Only valid for Algo orders                                                                                                                                                                                                                                                                      |
| deviation     | double  | Yes      | Only valid for Algo                                                                                                                                                                                                                                                                             |
| remainingSize | double  | Yes      | Size left to be transacted                                                                                                                                                                                                                                                                      |
| originalSize  | double  | Yes      | Original order size                                                                                                                                                                                                                                                                             |
| positionMode      | string  | Yes      | Position mode<br/>ONE_WAY, HEDGE or ISOLATED                                                                                                                                                                                                                                                    |
| positionDirection | string  | Yes      | Position direction                                                                                                                                                                                                                                                                              |
| positionId        | string  | Yes      | Position id                                                                                                                                                                                                                                                                                     |

## Query Position Mode

> Response

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

Retrieve user's position mode

### Request Parameters

| Name               | Type    | Required | Description          |
| ---                | ---     | ---      | ---------------------|
| symbol             | string  | No       | Market symbol        |

### Response Content

| Name         | Type   | Required | Description                |
| ---          | ---    | ---      |----------------------------|
| symbol       | string | Yes      | Market symbol              |
| positionMode | string | Yes      | ONE_WAY, HEDGE or ISOLATED |

## Query User Initial Margin Percentage And Maintenance Margin Percentage

> Response

```json
[
    {
        "symbol": "ETH-PERP",
        "initialMarginPercentage": 0.01,
        "maintenanceMarginPercentage": 0.005
    },
    {
        "symbol": "BTC-PERP",
        "initialMarginPercentage": 0.01,
        "maintenanceMarginPercentage": 0.005
    }
]
```

`GET /api/v1/margin_setting`

Queries user's initial margin percentage and maintenance margin percentage. When no symbol is specified, margin percentage for all markets will be returned.

### Request Parameters

| Name               | Type    | Required | Description   |
| ---                | ---     | ---      | ---           |
| symbol             | string  | No       | Market symbol |

### Response Content

| Name                        | Type   | Required | Description                           |
| ---                         | ---    | ---      | ---                                   |
| symbol                      | string | Yes      | Market symbol                         |
| initialMarginPercentage     | double | Yes      | Current initial margin percentage     |
| maintenanceMarginPercentage | double | Yes      | Current maintenance margin percentage |