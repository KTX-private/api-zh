# Market Data Endpoints

## 测试连通性

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/ping`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/ping'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{}
```



## 获取服务器时间

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/time`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/time'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{"time":"1746777864508"}
```

## 获取币种信息

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/coins`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/coins'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "asset": "USDT", // 资产名称
    "valid_decimals": 8, // 资产精度
    "enable_transfer": 1, // 允许划转 [0: 不允许 | 1: 允许]
    "chains": [
      {
        "coin_symbol": "USDT", // 币种名称
        "chain_type": "Tron (TRC20)", // 网络
        "enable_withdraw": 1, // 允许提现 [0: 不允许 | 1: 允许]
        "enable_deposit": 1, // 允许充值 [0: 不允许 | 1: 允许]
        "original_decimals": 6 // 网络精度
      },
      {
        "coin_symbol": "eUSDT",
        "chain_type": "Ethereum (ERC20)",
        "enable_withdraw": 1, // 允许提现 [0: 不允许 | 1: 允许]
        "enable_deposit": 1, // 允许充值 [0: 不允许 | 1: 允许]
        "original_decimals": 6
      },
      {
        "coin_symbol": "bUSDT",
        "chain_type": "BNB Smart Chain (BEP20)",
        "enable_withdraw": 1, // 允许提现 [0: 不允许 | 1: 允许]
        "enable_deposit": 1, // 允许充值 [0: 不允许 | 1: 允许]
        "original_decimals": 18
      },
      {
        "coin_symbol": "sUSDT",
        "chain_type": "Solana",
        "enable_withdraw": 1, // 允许提现 [0: 不允许 | 1: 允许]
        "enable_deposit": 1, // 允许充值 [0: 不允许 | 1: 允许]
        "original_decimals": 6
      }
    ]
  } 
]
```

**获取币种信息**

* 请求方式 GET
* 请求路径 /v1/coins
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明 |
|---------- | ---------- | ---------- |-----------------------------------------------------------------------------------------------------|

## 获取交易对

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/products?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/products?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[{
	"feeMode": 0, // 手续费模式[0：统一费率]
	"symbol": "BTC_USDT", // 交易对
	"takerFee": "0.002", // Taker费率
	"minOrderSize": "0.00001", // 最小下单数量
	"mini": 0, // 是否迷你合约[0：否丨1：是]
	"quantityIncrement": "0.000001", // 数量最小增量（步长）
	"profitSharing": "0", // 分润比例
	"priceIncrement": "0.01", // 价格最小增量（步长）
	"active": 1, // 是否启用[0：禁用丨1：启用]
	"maxOrderValue": "10000000000", // 最大下单价值
	"market": "spot", // 交易对市场[spot：现货丨lpc：U本位合约| forecast: 预测市场]
	"followFundingRate": 0, // 是否跟随资金费率[0：否丨1：是]
	"makerFee": "0.001", // Maker费率
	"quantityScale": 6, // 数量精度
	"priceScale": 2, // 价格精度
	"maxOrderSize": "10000", // 最大下单数量
	"fundingRateEx": "0", // 资金费率（扩展字段）
	"id": 1, // ID
	"time": "1781685210520", // 时间戳（毫秒）
	"minOrderValue": "10" // 最小下单价值
}]
```

**获取交易对列表**

* 请求方式 GET
* 请求路径 /v1/products
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                     |
|--------| ---------- |------|--------------------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等,不指定返回全部交易对 |

* Data source

  Cache

## 获取合约交易对杠杆和档位信息

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/pu/getPositionTierRules`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/pu/getPositionTierRules'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "EOS_USDT_SWAP": [
    {
      "no": 1,    // 档位
      "positionValue": "1000000", // 仓位价值<= 1000000
      "maintainMarginRate": "0.005", // 维持保证金率
      "initMarginRate": "0.01", // 初始保证金率
      "maxLeverage": "100" // 最大杠杆倍数
    },
    {
      "no": 2,
      "positionValue": "2000000",
      "maintainMarginRate": "0.0075",
      "initMarginRate": "0.015",
      "maxLeverage": "67"
    },
    ...
  ],
  "BTC_USDT_SWAP": [
    ...
  ],
  "ETH_USDT_SWAP": [
    ...
  ],
  ...
}
```

**获取合约交易对杠杆和档位信息**

* 请求方式 GET
* 请求路径 /v1/pu/getPositionTierRules



## 获取订单簿

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/order_book?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/order_book?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "i": 1027024, // Update ID
  "t": "1644558642100", // Update time
  "b": [ // 买单盘
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ]
    ...
  ],
  "a": [ // 卖单盘
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ],
    [
      "46125.7", // 委托价格
      "0.079045" // 委托量
    ]
    ...
  ]
}
```

**获取深度数据**

* 请求方式 GET
* 请求路径 /v1/order_book
* 请求参数


| 参数名称    | 参数类型 | 是否必传 | 说明                                                                                                                                                                                                                                                                                                                             |
| ------------- | ---------- | ---------- |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol      | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等                                                                                                                                                                                                                                                                                    |
| level       | int32    | 否       | 指定最多返回多少级深度<br/>有效值 1, 2, 5, 10, 20, 50, 100, 200, 500, 1000<br/>默认值 100                                                                                                                                                                                                                                                       |
| price_scale | integer  | 否       | 价格精度合并 [0: 4位小数；1: 3位小数；2: 2位小数；3: 1位小数；4: 0位小数] |

> 注意: 数据按价格最优排序, 即买侧深度按价格由大到小排序, 卖侧深度按价格由小到大排序

* Data source

  Cache

## 获取K线

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/candles?market=spot&symbol=BTC_USDT&time_frame=1m`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/candles?market=spot&symbol=BTC_USDT&time_frame=1m'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
  "t": 60000, // K线周期(ms)
  "e": [
    [
      "1644224940000", // 开盘时间
      "10190.53", // 开盘价
      "10192.5", // 最高价
      "9806.82", // 最低价
      "10127.37", // 收盘价
      "0.834", // 成交量
      "8370.40506", // 成交价值
      "1", // 首笔成交ID
      278 // 区间内总成交次数
    ],
    [
      "1644224940000",
      "10190.53",
      "10192.5",
      "9806.82",
      "10127.37",
      "0.834",
      "8370.40506",
      "1",
      278
    ],
    ...
  ]
}
```

**获取K线数据**

* 请求方式 GET
* 请求路径 /v1/candles
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                     |
| ------------ | ---------- | --------- |------------------------------------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol     | string   | 是      | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等                            |
| time_frame | string   | 是      | K线周期 [1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d, 1W, 1M] |
| before     | int64    | 否      | utc时间<br/>限定返回K线记录的最近时间                                                |
| after      | int64    | 否      | utc时间<br/>限定返回K线记录的最早时间                                                |
| limit      | integer  | 否      | 获取K线记录的最大数量<br/>默认值100,最大值1000                                         |

* 该接口支持的参数组合和数据源

  1. market + symbol + time_frame  --> cache
  2. market + symbol + time_frame + limit  --> cache
  3. market + symbol + time_frame + before  --> database
  4. market + symbol + time_frame + before + limit  --> database
  5. market + symbol + time_frame + after  --> database
  6. market + symbol + time_frame + after + limit  --> database

> 返回结果按时间由早及近排序

## 获取成交记录

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/trades?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/trades?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "i": 17122255, // 交易ID
    "p": "46125.7", // 成交价格
    "q": "0.079045", // 成交量
    "s": 1, // Taker方向 [1: 买方吃单 | -1: 卖方吃单]
    "t": "1628738748319" // 成交时间
  },
  ...
]
```

**获取交易记录**

* 请求方式 GET
* 请求路径 /v1/trades
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                          |
| ------------ | ---------- | ---------- |---------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol     | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等 |
| start_time | int64    | 否       | 限定返回交易记录的最早时间                               |
| end_time   | int64    | 否       | 限定返回交易记录的最近时间                               |
| before     | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最大id                   |
| after      | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最小id      |
| limit      | integer  | 否       | 获取记录的最大数量<br/>默认值100,最大值1000                |

* 该接口支持的参数组合和数据源

  1. market + symbol  --> cache
  2. market + symbol + limit  --> cache
  3. market + symbol + start_time  --> database
  4. market + symbol + start_time + limit  --> database
  5. market + symbol + end_time  --> database
  6. market + symbol + end_time + limit  --> database
  7. market + symbol + start_time + end_time  --> database
  8. market + symbol + start_time + end_time + limit  --> database
  9. market + symbol + before  --> database
  10. market + symbol + before + limit  --> database
  11. market + symbol + after  --> database
  12. market + symbol + after + limit  --> database

  *数据源为cache的参数组合用于获取最近1000条交易记录*

  *数据源为database的参数组合用于获取较早的交易记录*

  *如果用数据源为database的参数组合获取最新交易记录,其结果要比cache数据源稍有延迟*
* Usage
  **用法举例：获取三个月内某交易对的全部交易记录**

  1. 首先使用symbol + limit 参数组合获取最新的交易记录
  2. 将取到的首条记录的tradeId作为before参数的值,反复使用symbol + before + limit参数组合获取更多记录,直至获取三个月内的全部交易记录后停止

> 返回结果按交易记录id由小到大排序

## 获取 Tickers

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/ticker?market=spot&symbol=BTC_USDT`
request.get(url,
        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
          }

          console.log(body)

        });
```

```python
import requests

END_POINT = 'https://api.ktx.com/api';

def do_request():
    path = '/v1/ticker?market=spot&symbol=BTC_USDT'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
[
  {
    "askPrice": "98100", // 卖一价
    "product": "BTC_USDT", // 交易对
    "amount": "922635", // 24h成交价值
    "last": "98000", // 最新成交价
    "firstTradeId": 1, // 首笔成交ID
    "change": "0", // 24h价格变化
    "bidQty": "1.7", // 买一数量
    "bidPrice": "98000", // 买一价
    "volume": "9.41", // 24h成交量
    "lastQty": "0.3", // 最近一笔成交量
    "askQty": "0.5", // 卖一数量
    "high": "98100", // 24h最高价
    "tradeCount": 30, // 24h成交次数
    "low": "98000", // 24h最低价
    "time": "1733474204000", // 时间
    "open": "98000" // 开盘价格
  }
]
```

**获取ticker**

* 请求方式 GET
* 请求路径 /v1/ticker
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                  |
| ---------- | ---------- | ---------- |-----------------------------------------------------------------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol   | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等,<br/>可按如下两种形式指定多个交易对代码<br/> 1. symbol=BTC_USDT,ETH_USDT |

* Data Source

Cache

