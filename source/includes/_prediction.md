# 预测市场

## 获取 Event

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/forecast/events`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/forecast/events'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "endDate": "1781204400000", // 结束时间
    "description": "This event is for the upcoming FIFA World Cup game, scheduled for Thursday, June 11, 2026 between Mexico and South Africa.", // 描述
    "id": "351715", // id
    "title": "Mexico vs. South Africa", // 标题
    "slug": "fifwc-mex-rsa-2026-06-11", // slug
    "startDate": "1775515727000", // 开始时间
    "status": "active" // 市场状态 status [active:激活 ｜completed:完成]
  }
  ...
]
```

**获取events**

* 请求方式 GET
* 请求路径 /v1/forecast/events

## 获取 Events详情

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/forecast/event/detail`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/forecast/event/detail'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
  
    "endDate": "1781204400000", // 结束时间
    "description": "This event is for the upcoming FIFA World Cup game, scheduled for Thursday, June 11, 2026 between Mexico and South Africa.", // 描述
    "id": "351715", // id
    "title": "Mexico vs. South Africa", // 标题
    "slug": "fifwc-mex-rsa-2026-06-11", // slug
    "startDate": "1775515727000", // 开始时间
    "status": "active" // 市场状态 [active:激活 ｜completed:完成]
    "markets": [ // 可能包含多个market
      {
        "eventId": "351715", // event id
        "symbol": "1897034_FORECAST", // 交易对
        "question": "Will Mexico win on 2026-06-11?", // market 问题描述
        "takerFee": "0", // taker 手续费率
        "endDate": "1781204400000", // 结束时间
        "outcomes": [ // 问题回答选项
          "Yes", // 对应下单的long参数
          "No"   // 对应下单的short参数
        ],
        "winningOutcome": 0, // [0:未结算 |1:yes(long)赢 |-1:no(short)赢 |2:平局]
        "makerFee": "0.00040000", // maker 手续费率
        "quantityScale": 0, // 数量精度
        "priceScale": 2, // 价格精度
        "id": "1897034", // market id
        "slug": "fifwc-mex-rsa-2026-06-11-mex", // slug
        "startDate": "1775515619000", // 开始时间
        "status": "active" //  市场状态 [active:激活 ｜resolved:结算完成]
      }
      ...
    ],
  }
  ...
]
```

**获取events详情**

* 请求方式 GET
* 请求路径 /v1/forecast/event/detail
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明          |
|------| ---------- |------|-------------|
| id    | string   | 是    | 指定的event id |

## 获取 market

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/forecast/markets`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/forecast/markets'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "id": "1897034", // market id
    "eventId": "351715", // 关联的event id
    "slug": "fifwc-mex-rsa-2026-06-11-mex", // 市场唯一标识符
    "symbol": "1897034_FORECAST", // 交易对符号
    "question": "Will Mexico win on 2026-06-11?", // 预测问题 [格式: Will {事件主体} {结果} on {日期}?]
    "outcomes": [ // 预测结果选项
      "Yes", // 是/赢 [对应下单的long参数]
      "No"   // 否/输 [对应下单的short参数]
    ],
    "winningOutcome": 0, // 结算结果 [0: 未结算 | 1: Yes赢 | -1: No赢 | 2: 平局]
    "status": "active", //  市场状态 [active:激活 ｜resolved:结算完成]
    "startDate": "1775515619000", // 开始交易时间 [毫秒时间戳]
    "endDate": "1781204400000", // 结束/结算时间 [毫秒时间戳]
    "takerFee": "0", // taker手续费率 [吃单方手续费比例]
    "makerFee": "0.00040000", // maker手续费率 [挂单方手续费比例]
    "quantityScale": 0, // 数量精度 [小数位数，0表示整数]
    "priceScale": 2 // 价格精度 [小数位数，2表示保留2位小数]
  }
]
```

**获取market**

* 请求方式 GET
* 请求路径 /v1/forecast/markets
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                         |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| eventId  | string   | 否       | 指定的event id                                               |
| symbol   | string   | 否       | 预测市场交易对代码，格式为 `{marketId}_FORECAST`，如 `1897034_FORECAST`、`2362124_FORECAST` |

## 获取 ticker

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/ticker/get_all?market=forecast&forecastSide=yes`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/ticker/get_all?market=forecast&forecastSide=yes'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
      "productId": 759, // 交易对ID
      "product": "2434164_FORECAST", // 交易对符号
      "time": "1780644690000", // 时间戳 [毫秒]
      "last": "0.2235", // 最新成交价 [预测市场价格，范围0-1，表示事件发生概率]
      "lastQty": "15.1", // 最近一笔成交量 [合约张数]
      "bidPrice": "0.2234", // 买一价 [最高买入价格，表示买方认为事件发生概率]
      "bidQty": "4.7", // 买一数量 [买单委托张数]
      "askPrice": "0.2237", // 卖一价 [最低卖出价格，表示卖方认为事件发生概率]
      "askQty": "4.7", // 卖一数量 [卖单委托张数]
      "open": "0.2435", // 24h开盘价 [24小时前价格]
      "high": "0.2440", // 24h最高价 [最高概率]
      "low": "0.2227", // 24h最低价 [最低概率]
      "change": "-0.0818", // 24h价格变化 [正数:上涨 | 负数:下跌，表示概率变化绝对值]
      "volume": "112068.8", // 24h成交量 [合约张数]
      "amount": "26181.59142", // 24h成交额 [计价货币金额，如USDT]
      "tradeCount": 9496, // 24h成交次数 [撮合次数]
      "firstTradeId": 194285 // 首笔成交ID
}
```

**获取ticker**

* 获取的是yes价格
* 请求方式 GET
* 请求路径 /v1/ticker/get_all
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                         |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| market   | string   | 是       | 交易对市场 [forecast:预测市场]                               |
| forecastSide | string   | 否       | 盘口方向（yes: yes方向价格；no: no方向价格 ），默认值 yes                                                                   |

> 注：没有forecastSide参数时获取的ticker默认是以 Yes(long) 方向的价格展示

## 获取订单簿

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/order_book?market=forecast&symbol=2362124_FORECAST&forecastSide=no`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('upload failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/order_book?market=forecast&symbol=2362124_FORECAST&forecastSide=no'
    resp = requests.get(END_POINT + path)
    print(resp.text)

if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": 1027024, // Update ID [订单簿版本号，每次变更递增]
  "t": "1644558642100", // Update time [更新时间戳，毫秒]
  "b": [ // 买单盘 [买方委托队列，按价格从高到低排序，表示愿意买入的概率价格]
    [
      "0.65", // 委托价格 [买方愿意支付的价格，代表认为事件发生概率65%]
      "100.5" // 委托量 [买入合约张数]
    ],
    [
      "0.64", // 委托价格 [买方愿意支付的价格，代表认为事件发生概率64%]
      "200.3" // 委托量 [买入合约张数]
    ],
    [
      "0.63", // 委托价格 [买方愿意支付的价格，代表认为事件发生概率63%]
      "150.2" // 委托量 [买入合约张数]
    ]
  ],
  "a": [ // 卖单盘 [卖方委托队列，按价格从低到高排序，表示愿意卖出的概率价格]
    [
      "0.66", // 委托价格 [卖方愿意接受的价格，代表认为事件发生概率66%]
      "80.4" // 委托量 [卖出合约张数]
    ],
    [
      "0.67", // 委托价格 [卖方愿意接受的价格，代表认为事件发生概率67%]
      "120.6" // 委托量 [卖出合约张数]
    ],
    [
      "0.68", // 委托价格 [卖方愿意接受的价格，代表认为事件发生概率68%]
      "90.1" // 委托量 [卖出合约张数]
    ]
  ]
}
```

**获取订单簿**

* 获取的是yes价格
* 请求方式 GET
* 请求路径 /v1/order_book
* 请求参数

| 参数名称         | 参数类型 | 是否必传 | 说明                                                                                                       |
|:-------------| :------- | :------- |:---------------------------------------------------------------------------------------------------------|
| market       | string   | 否       | 交易对市场 [forecast: 预测市场]                                                                                   |
| symbol       | string   | 是       | 预测市场交易对代码，格式为 `{marketId}_FORECAST`，如 `1897034_FORECAST`、`2362124_FORECAST`                              |
| level        | int32    | 否       | 最大返回深度档位。批量查询时可使用 `symbol=1897034_FORECAST,1897035_FORECAST`。有效值：1、2、5、10、20、50、100、200、500、1000，默认值 100 |
| price_scale  | integer  | 否       | 价格精度合并。0=4位小数，1=3位小数，2=2位小数，3=1位小数，4=0位小数。默认值 0                                                          |
| forecastSide | string   | 否       | 盘口方向（yes: yes方向价格；no: no方向价格 ），默认值 yes                                                                   |

> 注：没有forecastSide参数时获取的订单簿默认是以 Yes(long) 方向的价格展示


## 创建预测市场订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

// 以80%的概率买入yes获胜
const param = {
    market:'forecast',
    symbol:'2362124_FORECAST',
    forecastSide:'buy_long', // 买yes获胜
    quantity:'1',
    price:'0.8', // 80%概率yes获胜
    type:'limit',
}

/*
// 以80%的概率买入no获胜
const param = {
    market:'forecast',
    symbol:'2362124_FORECAST',
    forecastSide:'buy_short', // 买no获胜
    quantity:'1',
    price:'0.8', // 80%概率no获胜
    type:'limit',
}
 */


let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/order`;

request.post({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });

```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'


def do_request():

    param = {
      'market':'forecast',
      'symbol':'2362124_FORECAST',
      'forecastSide':'buy_long',
      'quantity':'1',
      'price':'0.8',
      'type':'limit',
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/order'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.post(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "update_id": "1125900141912261326", // 更新ID
  "fees": [
    {
      "amount": "0.0354", // 资产数量
      "asset": "USDT", // 资产代码
      "value": "0" // 估值
    }
  ],
  "executedQty": "3", // 已成交数量
  "orderId": "4620329279333336924", // 订单ID
  "origin": 1, // 订单来源
  "type": "limit", // 订单类型 [limit: 限价 | market: 市价]
  "executedCost": "1.77", // 已成交成本（USDT）
  "forecastSide": "buy_long", // 预测方向 [buy_long: 买入yes；buy_short: 买入no；sell_long: 卖出yes；sell_short: 卖出now]
  "price": "0.59", // 委托价格（概率，范围 0-1）
  "timeInForce": "fok", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
  "introduction": "Bitcoin Up or Down - June 6, 3AM ET", // 事件描述
  "fills": [
    {
      "fees": [
        {
          "amount": "0.0354", // 资产数量
          "asset": "USDT", // 资产代码
          "value": "0" // 估值
        }
      ],
      "quantity": "3", // 成交数量（合约张数）
      "price": "0.59", // 成交价格（概率，范围 0-1）
      "time": "1780732741150", // 成交时间
      "taker": true, // 是否为Taker [true: 是 | false: 否]
      "profit": "0", // 收益
      "tradeId": 203 // 交易序号
    }
  ],
  "brokerId": 0, // 渠道ID
  "product": "2434164_FORECAST", // 交易对代码，格式为 {marketId}_FORECAST
  "quantity": "3", // 委托数量（合约张数）
  "stf": "disabled", // 订单成交转换方式 [disabled: 禁用自成交保护 | dc: 减少并取消 | co: 取消老订单 | cn: 取消新订单 | cb: 双向取消]
  "clientOrderId": "", // 自定义订单ID
  "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
  "updateTime": "1780732741150", // 更新时间
  "postOnly": false, // 只做maker [true: 是 | false: 否]
  "market": "forecast", // 市场类型 [forecast: 预测市场]
  "createTime": "1780732741150", // 创建时间
  "visibleQty": "-1", // 可见委托数量
  "fillCount": 1, // 成交次数
  "status": "filled" // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
}
```

**创建预测市场订单**

* 请求方式 POST
* 请求路径 /v1/order
* 权限: Trade
* 请求参数

| 参数名称        | 参数类型 | 是否必传 | 说明                                                                     |
| --------------- | -------- | -------- |------------------------------------------------------------------------|
| market          | string   | 是       | 市场类型 [forecast: 预测市场]                                                  |
| symbol          | string   | 是       | 交易对代码，如 2362124_FORECAST 等                                             |
| type            | string   | 是       | 委托类型 [limit: 限价；market: 市价]                                            |
| quantity        | decimal  | 是       | 委托量                                                                    |
| forecastSide    | string   | 是       | 预测方向 [buy_long: 买入yes；buy_short: 买入no；sell_long: 卖出yes；sell_short: 卖出now] |
| client_order_id | string   | 否       | 委托ID，有效值为int64整数的字符串，建议使用提交委托时的Unix时间戳                                 |
| price           | decimal  | 否       | 委托限价 范围为(0,1)开区间 0.8代表认为80%概率获胜                                        |
| time_in_force   | string   | 否       | 委托时效性 [gtc: 成交前有效；ioc: 即时剩余取消；fok: 全数成交否则取消]，默认值: gtc                  |


## 获取订单详情

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'id=4611772879845982339';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/order?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'

def do_request():
    path = '/v1/order'
    query_str = 'id=14118828812271651'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "update_id": "1125900141912261326", // 更新ID
  "fees": [
    {
      "amount": "0.0354", // 资产数量
      "asset": "USDT", // 资产代码
      "value": "0" // 估值
    }
  ],
  "executedQty": "3", // 已成交数量
  "orderId": "4620329279333336924", // 订单ID
  "origin": 1, // 订单来源
  "type": "limit", // 订单类型 [limit: 限价 | market: 市价]
  "executedCost": "1.77", // 已成交成本（USDT）
  "forecastSide": "buy_long", // 预测方向 [buy_long: 买入yes；buy_short: 买入no；sell_long: 卖出yes；sell_short: 卖出now]
  "price": "0.59", // 委托价格（概率，范围 0-1）
  "timeInForce": "fok", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
  "introduction": "Bitcoin Up or Down - June 6, 3AM ET", // 事件描述
  "fills": [
    {
      "fees": [
        {
          "amount": "0.0354", // 资产数量
          "asset": "USDT", // 资产代码
          "value": "0" // 估值
        }
      ],
      "quantity": "3", // 成交数量（合约张数）
      "price": "0.59", // 成交价格（概率，范围 0-1）
      "time": "1780732741150", // 成交时间
      "taker": true, // 是否为Taker [true: 是 | false: 否]
      "profit": "0", // 收益
      "tradeId": 203 // 交易序号
    }
  ],
  "brokerId": 0, // 渠道ID
  "product": "2434164_FORECAST", // 交易对代码，格式为 {marketId}_FORECAST
  "quantity": "3", // 委托数量（合约张数）
  "stf": "disabled", // 订单成交转换方式 [disabled: 禁用自成交保护 | dc: 减少并取消 | co: 取消老订单 | cn: 取消新订单 | cb: 双向取消]
  "clientOrderId": "", // 自定义订单ID
  "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
  "updateTime": "1780732741150", // 更新时间
  "postOnly": false, // 只做maker [true: 是 | false: 否]
  "market": "forecast", // 市场类型 [forecast: 预测市场]
  "createTime": "1780732741150", // 创建时间
  "visibleQty": "-1", // 可见委托数量
  "fillCount": 1, // 成交次数
  "status": "filled" // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
}
```

**获取指定 id 的委托**

* 请求方式 GET
* 请求路径 /v1/order
* 权限: View
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                |
| ---------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| id       | string   | 是       | 委托ID<br/>委托ID可以是交易所分配的, <br/>也可以是用户自定义的 (在提交委托时使用client_order_id参数).<br/>当使用自定义ID时, 需要在id前添加 “c:” 前缀.<br/>例如: 提交委托时使用了自定义ID “123”, 在获取委托时, 需使用 “c:123”. |

## 获取历史订单列表

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'limit=2&market=forecast&symbol=2434164_FORECAST';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/history/orders?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'

def do_request():
    path = '/v1/history/orders'
    query_str = 'limit=2&market=forecast&symbol=2434164_FORECAST'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "update_id": "1125900141912261326", // 更新ID
    "fees": [
      {
        "amount": "0.0354", // 资产数量
        "asset": "USDT", // 资产代码
        "value": "0" // 估值
      }
    ],
    "executedQty": "3", // 已成交数量
    "orderId": "4620329279333336924", // 订单ID
    "origin": 1, // 订单来源
    "type": "limit", // 订单类型 [limit: 限价 | market: 市价]
    "executedCost": "1.77", // 已成交成本（USDT）
    "forecastSide": "buy_long", // 预测方向 [buy_long: 买入yes；buy_short: 买入no；sell_long: 卖出yes；sell_short: 卖出now]
    "price": "0.59", // 委托价格（概率，范围 0-1）
    "timeInForce": "fok", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
    "introduction": "Bitcoin Up or Down - June 6, 3AM ET", // 事件描述
    "fills": [
      {
        "fees": [
          {
            "amount": "0.0354", // 资产数量
            "asset": "USDT", // 资产代码
            "value": "0" // 估值
          }
        ],
        "quantity": "3", // 成交数量（合约张数）
        "price": "0.59", // 成交价格（概率，范围 0-1）
        "time": "1780732741150", // 成交时间
        "taker": true, // 是否为Taker [true: 是 | false: 否]
        "profit": "0", // 收益
        "tradeId": 203 // 交易序号
      }
    ],
    "brokerId": 0, // 渠道ID
    "product": "2434164_FORECAST", // 交易对代码，格式为 {marketId}_FORECAST
    "quantity": "3", // 委托数量（合约张数）
    "stf": "disabled", // 订单成交转换方式 [disabled: 禁用自成交保护 | dc: 减少并取消 | co: 取消老订单 | cn: 取消新订单 | cb: 双向取消]
    "clientOrderId": "", // 自定义订单ID
    "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
    "updateTime": "1780732741150", // 更新时间
    "postOnly": false, // 只做maker [true: 是 | false: 否]
    "market": "forecast", // 市场类型 [forecast: 预测市场]
    "createTime": "1780732741150", // 创建时间
    "visibleQty": "-1", // 可见委托数量
    "fillCount": 1, // 成交次数
    "status": "filled" // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
  }
  ...
]
```

**获取历史订单列表**
1三个月内的已结算委托, 含已拒绝, 已撤销和已成交委托
2全部已成交委托
3全部已撤销的部分成交委托

* 请求方式 GET
* 请求路径 /v1/history/orders
* 权限: View


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                                  |
| ------------ | ---------- |------|-------------------------------------------------------------------------------------------------------------------------------------|
| market | string   | 是    | 交易对市场 [forecast: 预测市场]                                                                                                    |
| symbol     | string   | 否    | 交易对代码，如 2362124_FORECAST 等 <br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |
| start_time | long     | 否    | 限定返回委托的最早创建时间                                                                                                                       |
| end_time   | long     | 否    | 限定返回委托的最近创建时间                                                                                                                       |
| before     | int64    | 否    | 委托更新ID<br/>限定返回委托的最大更新ID                                                                                                            |
| after      | int64    | 否    | 委托更新ID<br/>限定返回委托的最小更新ID                                                                                                            |
| limit      | long     | 否    | 指定最多返回多少个委托                                                                                                                         |

## 获取未成交订单列表

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'market=forecast&symbol=2434164_FORECAST';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/pending/orders?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'

def do_request():
    path = '/v1/pending/orders'
    query_str = 'market=forecast&symbol=2434164_FORECAST'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "update_id": "1125900141912261326", // 更新ID
    "fees": [],
    "executedQty": "0", // 已成交数量
    "orderId": "4620329279333336924", // 订单ID
    "origin": 1, // 订单来源
    "type": "limit", // 订单类型 [limit: 限价 | market: 市价]
    "executedCost": "0", // 已成交成本（USDT）
    "forecastSide": "buy_long", // 预测方向 [buy_long: 买入yes；buy_short: 买入no；sell_long: 卖出yes；sell_short: 卖出now]
    "price": "0.59", // 委托价格（概率，范围 0-1）
    "timeInForce": "fok", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
    "introduction": "Bitcoin Up or Down - June 6, 3AM ET", // 事件描述
    "fills": [],
    "brokerId": 0, // 渠道ID
    "product": "2434164_FORECAST", // 交易对代码，格式为 {marketId}_FORECAST
    "quantity": "3", // 委托数量（合约张数）
    "stf": "disabled", // 订单成交转换方式 [disabled: 禁用自成交保护 | dc: 减少并取消 | co: 取消老订单 | cn: 取消新订单 | cb: 双向取消]
    "clientOrderId": "", // 自定义订单ID
    "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
    "updateTime": "1780732741150", // 更新时间
    "postOnly": false, // 只做maker [true: 是 | false: 否]
    "market": "forecast", // 市场类型 [forecast: 预测市场]
    "createTime": "1780732741150", // 创建时间
    "visibleQty": "-1", // 可见委托数量
    "fillCount": 1, // 成交次数
    "status": "accepted" // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
  },
  ...
]
```

**获取未成交的委托**

* 请求方式 GET
* 请求路径 /v1/pending/orders
* 权限: View


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                    |
| ------------ | ---------- |------|-----------------------------------------------------------------------------------------------------------------------|
| market | string   | 是    | 交易对市场 [forecast: 预测市场]                                                                                     |
| symbol     | string   | 否    | 交易对代码,如 2434164_FORECAST 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |

## 取消订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    id:'14244173146202090'
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/order/delete`;

request.post({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'

def do_request():

    param = {
        'id': '14245272657638034'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/order/delete'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.post(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[n] // 取消数量
```

**撤销指定 id 的委托**

* 请求方式 POST
* 请求路径 /v1/order/delete
* 权限: Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                        |
| ---------- | ---------- | ---------- |-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| id       | string   | 是       | 委托ID,支持多个id拼接<br>委托ID可以是交易所分配的,<br/>也可以是用户自定义的（在提交委托时使用client_order_id参数）。<br>当使用自定义ID时,需要在id前添加 “c:” 前缀。<br/>例如：提交委托时使用了自定义ID “123”, 在撤销委托时,需使用 “c:123”。 |


## 获取仓位

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'market=forecast&symbol=2434164_FORECAST';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/positions?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime 

        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'


def do_request():
    path = '/v1/positions'
    query_str = 'market=forecast&symbol=2434164_FORECAST'
    expire_time = str(int(time.time() * 1000) + 5000)
    # POST or DELETE replace query_str with body_str
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "entryPrice": "0.6", // 开仓价格
    "symbol": "2434164_FORECAST", // 交易代码
    "leverage": "1", // 杠杆倍数（预测市场固定为 1）
    "maintMargin": "0.0050000000", // 维持保证金率
    "side":"short", // 预测市场方向 [long: yes选项 | short: no选项]
    "quantity": "0.100", // 仓位数量 空仓0.1
    "posMargin": "0.6", // 保证金
    "marginMethod": "isolate", // 保证金模式（预测市场固定为 isolate）
    "closableQty": "0.100", // 可平仓位数量
    "initMargin": "0.1000000000", // 初始保证金率
    "id": "1125899906842624158", // ID
    "orderMargin": "0",  // 委托保证金
    "mergeMode": "short",  // 仓位合并模式 [long: 合并多仓 | short: 合并空仓]
    "introduction": "Bitcoin Up or Down - June 6, 3AM ET", // 事件描述
  }
  ...
]
```

**获取仓位**

* 请求方式 GET
* 请求路径 /v1/positions
* 权限: View
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                    |
| ------------ | ---------- |------|-----------------------|
| position_id   | string   | 否    | 仓位ID 如存在这个参数,优先级最高    |
| market | string   | 是    | 交易对市场 [forecast: 预测市场] |
| symbol     | string   | 否    | 配合market使用,交易对代如2434164_FORECAST等     |

## MergeSplit

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    id:'14244173146202090'
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/splitMerge`;

request.post({
        url:url,
        body:param,
        json:true,
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import json
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'

def do_request():

    param = {
        'id': '14245272657638034'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/splitMerge'
    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.post(END_POINT + path, json=param, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
""
```

**MergeSplit**

* 请求方式 POST
* 请求路径 /v1/splitMerge
* 权限: Trade
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                |
|------| ---------- | ---------- |-------------------|
| market | string   | 是   | 交易对市场 [forecast: 预测市场]                                            |
| symbol     | string   | 否       | 交易对代码<br/>如 2434164_FORECAST  |
| type | string   | 是       | split 或者 merge    |
| quantity        | decimal  | 是       | split 或者 merge的数量 |

## 获取成交明细

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'limit=2&market=forecast&symbol=2434164_FORECAST';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/fills?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':exprieTime 

        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('upload failed:', err);
        }
        console.log(body) // 7.the result

    });
```

```python
import hashlib
import hmac
import requests
import time

END_POINT = 'https://api.ktx.com/papi'
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'


def do_request():
    path = '/v1/fills'
    query_str = 'limit=2&market=forecast&symbol=2434164_FORECAST'
    expire_time = str(int(time.time() * 1000) + 5000)
    # POST or DELETE replace query_str with body_str
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "product": "2434164_FORECAST", // 交易对代码（预测市场格式为 {marketId}_FORECAST）
    "fees": [
      {
        "amount": "10", // 手续费数量
        "asset": "USDT", // 手续费资产代码
        "value": "10" // 手续费估值（USDT）
      }
    ],
    "quantity": "0.01", // 成交数量（现货/合约：基础币数量；预测市场：合约张数）
    "orderId": "4611772879845982371", // 订单ID
    "fillId": "1125899906842624338", // 成交记录ID
    "price": "1000000", // 成交价格（现货/合约：价格；预测市场：概率，范围 0-1）
    "time": "1733541360859", // 成交时间（毫秒时间戳）
    "taker": true, // 是否为Taker [true: 吃单方 | false: 挂单方]
    "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
    "tradeId": 26 // 交易序号
  }
]
```

**获取成交记录**

* 请求方式 GET
* 请求路径 /v1/fills
* 权限: View
* 请求参数(需要排序)


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                          |
| ------------ | ---------- | ---------- |-----------------------------------------------------------------------------|
| market | string   | 是   | 交易对市场 [forecast: 预测市场]                                            |
| order_id   | string   | 否       | 交易所分配的委托ID<br/>限定仅返回指定委托的成交记录<br/>如果不指定该参数,请指定 symbol                       |
| symbol     | string   | 否       | 交易对代码<br/>如 2434164_FORECAST 等<br/>限定仅返回指定交易对的成交记录<br/>如果不指定该参数,请指定 order_id |
| start_time | int64    | 否       | 限定返回成交记录的最早时间                                                               |
| end_time   | int64    | 否       | 限定返回成交记录的最近时间                                                               |
| before     | int64    | 否       | 成交记录 fillId<br/>限定返回成交记录的最大id                                               |
| after      | int64    | 否       | 成交记录 fillId<br/>限定返回成交记录的最小id                                               |
| limit      | int32    | 否       | 限定返回结果的最大条数<br/>默认值 100                                                     |

## 关于推送
* 在公开数据推送,订阅的格式只需要替换对应的market和symbol即可,返回的价格数据都是yes方向的价格,如order book的订阅：{"method":"SUBSCRIBE","params":["forecast.2434164_FORECAST.order_book.5"]}
* 在用户私有数据推送中,推送的的order 和 position和上面预测市场接口请求仓位和订单接口返回格式一致
