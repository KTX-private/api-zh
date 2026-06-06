---
title: Ktx API文档(v4)

language_tabs:
- javascript
- python

toc_footers:
- <a href='https://www.ktx.com'>开始交易</a>

includes:
- errors

search: true

code_clipboard: true

---
# 基本信息

## API

**使用API开发应用程序,您可以准确地获取Ktx现货市场的行情数据,快速进行自动化交易。API包含众多接口,按功能大致分为以下几组：**

* Market Data Endpoints 用于获取行情数据的REST接口
* User Data Endpoints 用于获取用户私有数据的REST接口
* Market Data Streams 用于获取行情数据的WebSocket接口
* User Data Streams 用于获取用户私有数据的WebSocket接口

**API使用如下Base URL：**

* Market Data Endpoints: https://api.ktx.com/api
* User Data Endpoints: https://api.ktx.com/papi (老版本的路径是以/api结尾的,现在已经不支持了,请务必使用/papi)
* Market Data Stream: wss://m-stream.ktx.com
* User Data Stream: wss://u-stream.ktx.com

**备用api域名列表**

[https://api.ktx.com/api/v1/pu/domains](https://api.ktx.com/api/v1/pu/domains)

**API的REST接口使用以下HTTP方法：**

* GET 用于获取行情或私有数据
* POST 用于提交委托等操作

**API的REST接口所需的参数应根据以下规则附加于请求中：**

* GET类型的接口参数应附加于Query String中
* POST类型的接口参数应以JSON格式附加于Request Body中

**Response**

API的响应数据都以JSON格式返回,具体格式请参考各接口的描述。

**Error**

API的错误以如下JSON格式返回：

{<br/>
&nbsp;&nbsp;"state": error code,<br/>
&nbsp;&nbsp;"msg": "error message"<br/>
}

*其中,state表示错误的类型,msg包含错误产生的原因或如何避免错误的提示。具体的错误类型请参考[Error](#errors)章节的内容。*

**Time or Timestamp**

API接口参数和响应数据中所涉及的时间值都是UNIX时间,单位为毫秒。

---

## 流量限制

**Ktx对来自于同一IP的请求做以下访问限制:**

1. Access Limits 访问频率限制
2. Usage Limits CPU用量限制

* Access Limits访问频率限制
  1. 同一IP每10秒最多10000次请求,超出限制的请求会收到-20007错误。
  2. 用户可以根据需要在10秒内以任意频率发送最多10000次请求,可以大约每10ms发送一次,也可以在1秒内连续发送10000次,然后等待9秒。
     <br/>
     <br/>
* Usage Limits用量限制
  1. 同一IP每10秒最多消耗10000点CPU 时间,超出限制的请求会收到-20006错误。
  2. 不同API消耗的CPU时间不同,这取决于API如何访问数据。
  3. 在本文中, 每个API接口访问数据的方式会以“缓存”, “数据库”的形式标明。访问缓存的API消耗的CPU时间较少,访问数据库的API消耗的CPU时间较多。根据用户发送的参数,API可能混合访问缓存和数据库,甚至多次访问数据库,这会增加API消耗的CPU时间。
  4. 每次API请求消耗的CPU时间会包含在响应头Ktx-Usage中,其格式为t1:t2:t3,其中,t1表示本次API请求消耗的CPU时间,t2表示最近10秒内当前IP消耗的CPU时间,t3表示最近10秒内当前IP剩余的可用CPU时间。

---

## 认证

> 完整例子

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'asset=BTC';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString(); // POST or DELETE  replace queryStr with bodyStr
const url = `${endpoints}/v1/trade/accounts?${queryStr}`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/trade/accounts'
    query_str = 'asset=BTC'
    expire_time = str(int(time.time() * 1000) + 5000)
    # POST or DELETE replace query_str with body_str
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time': expire_time
    }
    resp = requests.get(END_POINT + path, query_str, headers=headers)
    print(resp.text)


if __name__ == '__main__':
    do_request()
```

**身份验证**

* 私有接口用于访问账户、委托等私有信息,在请求时需要附加签名,以满足Ktx进行身份验证。本节将描述如何创建签名。

**生成Api Key**

* 要创建签名,首先需要生成Api Key和Secret Key组合。请牢记在此过程中生成的Secret Key,因为该值仅显示一次,如果忘记了Secret Key,请删除该Api Key,并生成新的Api Key和Secret Key组合。

**HTTP 请求头**

**访问私有接口的请求都必需附加以下HTTP请求头：**

* api-key 已生成的Api Key
* api-sign 签名

**如果需要,也可以附加以下HTTP请求头：**

* api-expire-time
  1. 接口过期时间。
  2. 该值是以毫秒为单位的Unix时间,服务器会忽略该时间之后收到的请求,这主要用于避免网络延迟带来的影响。

**创建签名**

在发送请求前,首先确定用于签名的消息体。对于GET类型的请求,Query String是需要签名的消息体,对于POST请求,Body String是需要签名的消息体,expire_time 是过期时间
签名的具体方法如下：

* 第一步：获取当前时间戳 + 有效时间
  使用 Date.now() 获取当前毫秒时间戳,并加上一个短暂的有效期（如 5000 毫秒）,表示请求将在 5 秒内失效。
* 第二步：计算签名
  将 [时间戳 + 请求体字符串] 作为原始数据,使用 HMAC-SHA256 算法,并使用用户的 secret 作为密钥进行加密：
* 第三步：将以上结果转化为Hex String
* 第四步：将Hex String作为请求头api-sign的值,将expire_time作为请求头api-expire-time的值

## Api Key权限

**私有接口需要特定的权限才能执行。可以为Api Key授予适当的权限。如果Api Key未被授予某个接口需要的权限,那么使用该Api Key提交的请求将被拒绝。**

**可以授予Api Key以下权限：**

* View权限允许Api Key获取私有数据。
* Trade权限允许Api Key提交或撤销委托,并允许Api Key获取交易相关的数据。

*接口需要的权限将在每个接口的描述中给出。*

---

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

END_POINT = 'https://api.ktx.com/time';

def do_request():
    path = '/v1/ping'
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
            return console.error('upload failed:', err);
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
[
  {
    "id": 5 // ID
    "market": "lpc", // 交易对市场 [spot: 现货 | lpc: U本位合约]
    "symbol": "BTC_USDT_SWAP", // 交易对
    "takerFee": "0.001", // Taker费率
    "makerFee": "0.001", // Maker费率
    "minOrderSize": "0.0001", // 最小下单数量
    "maxOrderSize": "10000000",// 最大下单数量
    "quantityScale": 4, // 数量精度
    "priceScale": 4, // 价格精度
    "minOrderValue": "0.0001", // 最小下单价值
    "maxOrderValue": "10000000000", // 最大下单价值
  }
]
```

**获取交易对列表**

* 请求方式 GET
* 请求路径 /v1/products
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                     |
|--------| ---------- |------|--------------------------------------------------------|
| market | string   |  否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                              |
| symbol | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等,不指定返回全部交易对 |

* Data source

  Cache

## 获取订单薄

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/order_book?market=spot&symbol=BTC_USDT`
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
| market | string   |  否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                              |
| symbol      | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等                                                                                                                                                                                                                                                                                    |
| level       | int32    | 否       | 指定最多返回多少级深度<br/>有效值 1, 2, 5, 10, 20, 50, 100, 200, 500, 1000<br/>默认值 100                                                                                                                                                                                                                                                       |
| price_scale | integer  | 否       | 价格精度合并 [0: 4位小数 | 1: 3位小数 | 2: 2位小数 | 3: 1位小数 | 4: 0位小数]<br/>默认值 0 |

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
            return console.error('upload failed:', err);
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
| market | string   |  否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                              |
| symbol     | string   | 是      | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等                            |
| time_frame | string   | 是      | K线周期 [1m | 3m | 5m | 15m | 30m | 1h | 2h | 4h | 6h | 12h | 1d | 1W | 1M] |
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
            return console.error('upload failed:', err);
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
| market | string   |  否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                              |
| symbol     | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等 |
| start_time | int64    | 否       | 限定返回交易记录的最早时间                               |
| end_time   | int64    | 否       | 限定返回交易记录的最近时间                               |
| before     | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最大id                   |
| after      | int64    | 否       | 交易记录 id<br/>限定返回交易记录的最大id                   |
| limit      | integer  | 否       | 获取记录的最大数量<br/>默认值100,最大值1000                |

* 该接口支持的参数组合和数据源

  1. market + symbol  --> cache
  2. market + symbol + limit  --> cache
  3. market + symbol + start_time  --> database
  4. market + symbol + start_time + limit  --> database
  5. market + symbol + end_time  --> database
  6. market + symbol + end_time + limit  --> database
  7. market + market + symbol + start_time + end_time  --> database
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
            return console.error('upload failed:', err);
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
| market | string   |  否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                              |
| symbol   | string   | 是       | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等,<br/>可按如下两种形式指定多个交易对代码<br/> 1. symbol=BTC_USDT,ETH_USDT |

* Data Source

Cache

# Market Data 推送

## Overview

> 例子

```javascript
const WebSocket = require('ws');
const ktxws = 'wss://m-stream.ktx.com';

let wsClass = function () {
};

wsClass.prototype._initWs = async function () {
    let that = this;
    console.log(ktxws)
    let ws = new WebSocket(ktxws);
    that.ws = ws;

    ws.on('open', function open() {
        console.log(new Date(), 'open')
        ws.send(JSON.stringify({"method":"SUBSCRIBE","params":["spot.BTC_USDT.order_book.5"]}));
        setInterval(function () {
          ws.ping(Date.now())
        },30000)
    });

    ws.on('close', data => {
        console.log('close, ', data);
    });

    ws.on('error', data => {
        console.log('error', data);
    });

    ws.on('ping', data => {
        console.log('ping ', data.toString('utf8'));
    });

    ws.on('pong', data => {
        console.log('pong ', data.toString('utf8'));
    });

    ws.on('message', data => {
        console.log("rece message")
        console.log(data)
    });
};

let instance = new wsClass();

instance._initWs().catch(err => {
    console.log(err);
});

```

```python
import websocket
import json
import threading
import time
from datetime import datetime

ws_url = 'wss://m-stream.ktx.com'

def stringify(obj):
    return json.dumps(obj, sort_keys=True).replace("\'", "\"").replace(" ", "")


def get_sub_str():
    subdata = {"method":"SUBSCRIBE","params":["spot.BTC_USDT.order_book.5"]} 
    return stringify(subdata)


def on_message(ws, message):
    print(message)


def on_error(ws, error):
    print(error)


def on_close(ws, close_status_code, close_msg):
    print("### closed ###")
    print("Status Code:", close_status_code)
    print("Message:", close_msg)

def ping_loop(ws):
  while True:
    time.sleep(30)

    data = {
      "ping": datetime.now().timestamp() * 1000
    }
    ws.send(json.dumps(data))
    
def on_open(ws):
    ws.send(get_sub_str())
    threading.Thread(target=ping_loop, args=(ws,), daemon=True).start()


def connect():
    # websocket.enableTrace(True)
    ws = websocket.WebSocketApp(ws_url,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever(ping_interval=0)


if __name__ == "__main__":
    connect()

```

**使用 Websocket 推送服务可以及时获取行情信息。**

* 连接 Websocket 服务器
  请使用以下 URL 连接 Websocket 服务器：
  <br/>
  wss://m-stream.ktx.com

> 在连接后,客户端可以发送以下JSON格式的请求给服务器

```json
{
  "id": 123, // 由客户端给定的请求ID
  "method":"SUBSCRIBE", // 请求类型
  "params":["spot.BTC_USDT.order_book.5"]
}
```

> 在收到请求后,服务器会发送以下JSON格式的响应给客户端

```json
{
  "result":"success", // 返回结果
  "op":"SUBSCRIBE", 
  "id": 123, // 由客户端给定的请求ID
  "events":["spot.BTC_USDT.order_book.5"]
}
```

> 如果发生错误,服务器会发送以下错误信息给客户端

```json
{
  "id": 123, // 请求ID
  "error": -1003, // 错误代码
  "message": "..." // 错误描述
}
```

> 同时,服务器还会发送以下JSON格式的数据流给客户端, 数据流包含市场行情的变化信息

```json
{
  "stream": "spot.BTC_USDT.order_book.5", // 数据流名称
  "data": ..., // 数据
}
```

> 请求：订阅数据流

```json
{
  "id": 1,
  "method": "SUBSCRIBE",
  "params": [
    "stream name",
    "stream name",
    ...
  ]
}
```

> 在连接后,请首先发送该请求给服务器,随后,服务器会在行情变化时发送相应的数据流给客户端。

> "data stream name" 是数据流名称,数据流名称是以下格式的字符串。
> market.symbol.data_type.param1.param2...

> 其中,market是交易对市场,如spot和lpc
> symbol是交易对名称,如 BTC_USDT、ETH_USDT 等。
> data_type是数据类型,目前仅支持以下数据类型
> order_book: 深度
> trades: 交易列表
> candles: K线
> ticker: 最新成交信息

> 在data_type之后是参数列表,不同的数据类型有不同的参数列表,这些将在后续介绍

> 请求: 取消订阅数据流

```json
{
  "id": 1,
  "method": "UNSUBSCRIBE",
  "params": [
    "data stream name",
    "data stream name",
    ...
  ]
}
```

> 如果请求被服务器正确处理,客户端会收到以下响应:

```json
  {
    "result":"success", // 返回结果
    "op":"SUBSCRIBE",
    ... 
    }
```

> 如果请求出错,客户端会收到以下错误响应:

```json
{
  "error": -1003, // 错误代码
  "message": "..." // 错误描述
}
```

## 请求类型及参数

> 请求类型及参数

> 客户端可以发送以下请求给服务器

```json
{
  "id": 123, // 由客户端给定的请求ID
  "method": "..." // 请求类型
  "params": [ // 请求参数列表
    "...",
    "...",
  ]
}
```

* 其中method字段的值是以下请求类型之一:


| 可选值      | 说明                                                             |
| ------------- |----------------------------------------------------------------|
| SUBSCRIBE   | 1.订阅数据流<br/> 2.参数是数据流名称列表 <br/> 3.在成功订阅后,服务器会在行情发生变化时发送数据流给客户端 |
| UNSUBSCRIBE | 1.取消订阅数据流<br/> 2.参数是数据流名称列表 <br/> 3.在成功取消订阅后,客户端不会再收到相应的数据流    |

## 订阅深度信息

**订阅深度信息**

> 发送以下请求可订阅深度信息

```json
{
  "id": 123,
  "method": "SUBSCRIBE",
  "params": [
    "spot.BTC_USDT.order_book.20",
    "spot.ETH_USDT.order_book.20",
    ...
  ]
}
```

* 参数

  1. 该请求的参数是深度流名称,格式如下：

  * \<market>\<symbol>.order_book.\<max_depth>
    1. \<market> 是交易对市场,如spot,lpc
    2. \<symbol> 是交易对名称,如BTC_USDT,ETH_USDT等
    3. \<max_depth> 是最大深度,有效值是5, 10, 20, 50, 100, 200, 500, 1000

> 数据流

```json
  {
    "stream": "spot.BTC_USDT.order_book.20",
    "data": {
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
          ],
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
          ],
       ...
      ]
    }
  }
```

> 在成功订阅后,客户端会首先收到一个完整深度的数据流,之后会收到增量变化数据流,请按以下方法合成完整的深度,或使用SDK.
>


## 订阅交易列表

**订阅交易列表**

> 发送以下请求可订阅交易列表

```json
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.trades",
    "spot.ETH_USDT.trades",
    ...
  ]
}
```

* 参数

  1. 该请求的参数是交易流名称,格式如下：

  * \<market>\<symbol>.trades
    1. \<market> 是交易对市场,如spot,lpc
    2. \<symbol> 是交易对名称,如BTC_USDT,ETH_USDT等

> 数据流

```json
{
  "stream": "spot.BTC_USDT.trades",
  "data": [
    {
      "i": 17122255, // 交易ID
      "p": "46125.7", // 成交价格
      "q": "0.079045", // 成交量
      "s": 1, // Taker方向 [1: 买方吃单 | -1: 卖方吃单]
      "t": "1628738748319" // 成交时间
    },
    ...
  ]
}
```

## 订阅K线

**订阅K线**

> 发送以下请求可订阅K线

```json
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.candles.1m",
    "spot.ETH_USDT.candles.1h",
    ...
  ]
}
```

* 参数

  1. K线流名称格式如下：

  * \<market>\<symbol>.candles.\<time_frame>
    1. \<market> 是交易对市场,如spot,lpc
    2. \<symbol> 是交易对名称,如BTC_USDT,ETH_USDT等
    3. \<time_frame> 是K线周期,有效值是1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 6h, 12h, 1d, 1w, 1M
> 数据流

```json
{
  "stream": "spot.BTC_USDT.candles.1m",
  "data": {
      "t":60000, // K线周期(ms)
      "e":[
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
      ...
    ]}
  }
}
```

## 订阅ticker

**订阅ticker**

> 发送以下请求可订阅Ticker

```json
{
  "id": 123,
  "method": "SUBSCRIBE"
  "params": [
    "spot.BTC_USDT.ticker",
    "spot.ETH_USDT.ticker",
    ...
  ]
}
```

* 参数

  1. Ticker流名称格式如下：

  * \<market>\<symbol>.ticker
    1. \<market> 是交易对市场,如spot,lpc
    2. \<symbol> 是交易对名称,如BTC_USDT,ETH_USDT等
> 数据流

```json
{
  "stream": "spot.BTC_USDT.ticker",
  "data": {   
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
}
```

---

# User Data Endpoints



## 获取交易账户资产

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret
const exprieTime = Date.now()+5000;

const queryStr = 'asset=BTC';
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/trade/accounts?${queryStr}`;

request.get(url,{
          headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time': exprieTime
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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/trade/accounts'
    query_str = 'asset=BTC'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time': expire_time
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
    "asset":"USDT",  // 资产代码
    "balance":"100",  // 总额
    "locked":"0",  // 冻结额
    "free":"100",  // 可用
    "withdrawable":"100",// 可转出额
    "collateral":false,// 是否抵扣保证金 [true: 是 | false: 否]
    "discountForMargin":"1", // 可选字段 保证金抵扣率 [0: 不可用 | 0.5: 50% | 1: 100%]
    "discountForFee":"1" // 可选字段 手续费抵扣率 [0: 不可用 | 0.5: 50% | 1: 100%]
  },
  {
    "asset":"BTC",  // 资产代码
    "balance":"100",  // 总额
    "locked":"0",  // 冻结额
    "free":"100",  // 可用
    "withdrawable":"100",// 可转出额
    "collateral":false,// 是否抵扣保证金 [true: 是 | false: 否]
  },
  ...
]
```

**获取 API Key 对应账户中各种资产的余额, 冻结等信息**

* 请求方式 GET
* 请求路径 /v1/trade/accounts
* 权限: View
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                                 |
| ---------- | ---------- | ---------- |----------------------------------------------------------------------------------------------------------------------------------------------------|
| asset    | string   | 否       | 资产代码,如 BTC, ETH 等<br/>可按以下两种形式指定多个资产代码<br/>1. accounts?asset=BTC,ETH<br/> 2. accounts?asset=BTC&asset=ETH <br/> 如果不指定 asset 参数, 则返回全部资产的信息 |

* Data Source

  Cache

## 获取充值地址

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    coin_symbol:'sUSDT'
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/depositAddr`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'coin_symbol': 'sUSDT'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/depositAddr'
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
[
  {
    "addr": "74VZm4a6eGGBW3VCq6umcPL8QPhs5fFnpcT9nyQGgHkw", // 地址
    "coin_id": "100", // 币种ID
    "coin_symbol": "sUSDT",// 币种
    "chain_type":"Solana", // 网络
    "general_name": "USDT" ,// 资产名称
    "mx_uid": "de4a8cdc-6be5-3253-9f34-d2c61a89286f"
  }
]
```

**获取充值地址**

* 请求方式 POST
* 请求路径 /v1/depositAddr
* 权限: View
* 请求参数


| 参数名称        | 参数类型 | 是否必传 | 说明                                          |
|-------------| ---------- |------|---------------------------------------------|
| coin_symbol | string   | 是    | 可以从/v1/coins 接口获取,具体链的币种名称如USDT,sUSDT,BTC 等 |

## 提现

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    coin_symbol:"BTC",
    amount:"0.001",
    addr:"bc1qksfjx5ezznnngk6grt04h8lwnta2mxtmjl0etm"
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/withdraw`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'coin_symbol':'BTC',
        'amount':'0.001',
        'addr':'bc1qksfjx5ezznnngk6grt04h8lwnta2mxtmjl0etm'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/withdraw'
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
[
  {
    "id": 624, // 提现ID
    "to_address": "bc1qksfjx5ezznnngk6grt04h8lwnta2mxtmjl0etm", // 到账地址
    "amount_real": "9.00000000", // 到账数量
    "amount": "10.00000000",  // 提现数量
    "fee": "1.00000000",  // 手续费
    "chain_type": "Bitcoin", //网络
    "coin_symbol": 'BTC' // 币种
  }
]
```

**提现**

* 请求方式 POST
* 请求路径 /v1/withdraw
* 权限: Withdraw
* 请求参数


| 参数名称        | 参数类型   | 是否必传 | 说明                                     |
|-------------|--------|-----|----------------------------------------|
| coin_symbol | string | 是   | 可以从/v1/coins 接口获取,具体链的币种名称如USDT,sUSDT,BTC 等 |
| addr        | string | 是   | 到账地址                                   |
| amount      | number | 是   | 提现数量                                   |
| memo        | string | 否   | memo备注                                 |
| withdraw_id | string | 否   | 自定义ID 最大长度为36 返回值会出现该字段用于幂等处理          |


## 获取钱包账户资产

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    asset:'USDT'
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/main/accounts`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'asset': 'USDT'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/main/accounts'
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
[
  {
    "asset": "USDT", //资产代码
    "balance": "100", // 总额
    "locked": "0"// 冻结额
    "free": "100"// 可用
  }
]
```

**获取钱包账户资产**

* 请求方式 POST
* 请求路径 /v1/main/accounts
* 权限: View
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                               |
|--------| ---------- |------|--------------------------------------------------|
| asset  | string   | 否    | 资产代码,如 BTC, ETH 等<br/>如果不指定 asset 参数, 则返回全部资产的信息 |


## 资产划转

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    symbol:'USDT',
    amount: 10,
    type: 'WALLET_TRADE',
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/transfer`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'symbol': 'USDT',
        'amount': 10,
        'type': 'WALLET_TRADE',
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/transfer'
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
  "asset": "BTC",
  "balance": "1.123"
}
```

**资产划转**

* 请求方式 POST
* 请求路径 /v1/transfer
* 权限: Trade
* 请求参数


| 参数名称        | 参数类型   | 是否必传 | 说明                                                      |
|-------------|--------|------|---------------------------------------------------------|
| symbol      | string | 是    | 资产代码,如 BTC, ETH 等                                       |
| amount      | number | 是    | 划转数量                                                    |
| type        | string | 是    | 划转类型 [WALLET_TRADE: 钱包→交易账户 | TRADE_WALLET: 交易账户→钱包] |
| transfer_id | string | 否   |自定义ID 最大长度为36 返回值会出现该字段用于幂等处理          |

## 子账户资产划转

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    'symbol':'BTC',
    'amount':'0.001',
    'sub_user_id':30000416,
    'side':'in',
    'transfer_id':'userdefineid001',
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/subaccount/transfer`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'symbol':'BTC',
        'amount':'0.001',
        'sub_user_id':30000416,
        'side':'in',
        'transfer_id':'userdefineid001',
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/subaccount/transfer'
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
  "asset": "BTC",
  "balance": "1.123"
}
```

**子账户资产划转**

* 请求方式 POST
* 请求路径 /v1/subaccount/transfer
* 权限: Trade
* 请求参数


| 参数名称        | 参数类型   | 是否必传 | 说明                            |
|-------------|--------|------|-------------------------------|
| symbol      | string | 是    | 资产代码,如 BTC, ETH 等             |
| amount      | number | 是    | 划转数量                          |
| sub_user_id      | number | 是    | 子账户ID                         |
| side        | string | 是    | 划转方向 [in: 主账户→子账户 | out: 子账户→主账户] |
| transfer_id | string | 否   | 自定义ID 最大长度为36 返回值会出现该字段用于幂等处理 |

## 获取账单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'asset=BTC&end_time=1651895799668&limit=10';
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + queryStr, secret).toString();
const url = `${endpoints}/v1/ledgers?${queryStr}`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/ledgers'
    query_str = 'asset=BTC&end_time=1651895799668&limit=10'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time': expire_time

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
    "amount":"10000",   // 变化数量
    "balance":"10000",  // 余额
    "id":"1125899906842624029", // ID
    "time":"1733468814795", // 时间
    "asset":"USDT",   // 资产代码
    "type":"transfer" // 账单类型
  }
  ...
]
```

**获取 API Key 对应账户的账单,包含一切改变账户余额的记录,如资金划转、交易、手续费收取等**

* 请求方式 GET
* 请求路径 /v1/ledgers
* 权限: View


| 参数名称       | 参数类型 | 是否必传 | 说明                                                                                                                                                 |
|------------| ---------- | ---------- |----------------------------------------------------------------------------------------------------------------------------------------------------|
| asset      | string   | 否       | 资产代码,如 BTC, ETH 等<br/>可按以下两种形式指定多个资产代码<br/>1. /v1/ledgers?asset=BTC,ETH<br/> 2. /v1/ledgers?asset=BTC&asset=ETH <br/> 如果不指定 asset 参数, 则返回全部资产的账单记录 |
| start_time | int64    | 否       | 限定返回账单记录的最早时间                                                                                                                                      |
| end_time   | int64    | 否       | 限定返回账单记录的最近时间                                                                                                                                      |
| before     | int64    | 否       | 账单记录id<br/>限定返回账单记录的最大id值                                                                                                                          |
| after      | int64    | 否       | 账单记录id<br/>限定返回账单记录的最小id值                                                                                                                          |
| limit      | int32    | 否       | 限定返回账单记录的最大条数<br/>默认值 100                                                                                                                          |
| type       | string   | 否       | 账单类型 [transfer: 划转 | trade: 交易 | fee: 手续费 | rebate: 返佣 | funding: 资金费用] |

* Data Source

  DB

## 创建订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    symbol:'BTC_USDT',
    side:'buy',
    quantity:'0.0001',
    price:'90000',
    type:'limit',
    market:'spot',
}


/* 
mini合约下单参数 
开多方向
const param = {
  symbol:'BTC_USDT_SWAP',
  side:'buy',
  quantity:'0.0001',
  type:'market',
  market:'lpc',
  leverage:20,
  mini:true,
  positionMerge:'none',
  marginMethod:'isolate'
}
平多方向
const param = {
  symbol:'BTC_USDT_SWAP',
  side:'sell',
  quantity:'0.0001',
  type:'market',
  market:'lpc',
  leverage:20,
  mini:true,
  positionMerge:'none',
  marginMethod:'isolate'
  positionId:1125899906842649789
  close:true
}

平仓的止盈止损
多仓平仓的止损
const param = {
  symbol:'BTC_USDT_SWAP',
  side:'sell',
  quantity:'0.0001',
  type:'stop', // 市价止损,如果使用stop-limit限价止损则需要增加price参数指定触发后的委托价格
  trigger_price:'50000', // 止损的触发价格 需要<当前价格
  market:'lpc',
  leverage:20,
  positionMerge:'long',
  marginMethod:'cross'
  positionId:1125899906842649789
  close:true
}
多仓平仓的止赢
const param = {
  symbol:'BTC_USDT_SWAP',
  side:'sell',
  quantity:'0.0001',
  type:'take-profit', // 市价止盈,如果使用take-profit-limit限价止盈则需要增加price参数指定触发后的委托价格
  trigger_price:'100000', // 止损的触发价格 需要>当前价格
  market:'lpc',
  leverage:20,
  positionMerge:'long',
  marginMethod:'cross'
  positionId:1125899906842649789
  close:true
}

开仓的同时设置止盈止损,以开多为例子

const param = {
  symbol:'BTC_USDT_SWAP',
  side:'buy',
  quantity:'0.0001',
  price:'73816.6',
  type:'limit', 
  market:'lpc',
  leverage:20,
  positionMerge:'long',
  marginMethod:'cross'
  close:false,
  tpo_trigger:1, // 开启止盈
  tpo_trigger_value:'80000', // 止盈触发价格 >当前价格
  slo_trigger:1, // 开启止损
  slo_trigger_value:'50000', // 止损触发价格 <当前价格
}

空仓止盈止损 trigger_price,tpo_trigger_value,slo_trigger_value的处理和多仓相反

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():

    param = {
      'symbol':'BTC_USDT',
      'side':'buy',
      'quantity':'0.0001',
      'price':'90000',
      'type':'limit',
      'market':'spot',
    }
    """
    mini合约下单参数 
    开多方向
    param = {
      'symbol':'BTC_USDT_SWAP',
      'side':'buy',
      'quantity':'0.0001',
      'type':'market',
      'market':'lpc',
      'leverage':20
      'mini':true
      'positionMerge':'none',
      'marginMethod':'isolate'
    }
    平多方向
    param = {
      'symbol':'BTC_USDT_SWAP',
      'side':'sell',
      'quantity':'0.0001',
      'type':'market',
      'leverage':20
      'market':'lpc',
      'mini':true
      'positionMerge':'none',
      'marginMethod':'isolate'
      'positionId':1125899906842649789
      'close':true
    }
    """
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
  "orderId": "4611767382287843330", // 订单ID
  "clientOrderId": "",  // 自定义订单ID
  "createTime": "1733390630904", // 创建时间
  "product": "BTC_USDT_SWAP", // 交易对代码
  "type": "limit", // 订单类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损]
  "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
  "quantity": "0.01", // 委托数量
  "stf": "disabled", // 订单成交转换方式 [0: disabled 禁用自成交保护 | 1: dc 减少并取消 | 2: co 取消老订单 | 3: cn 取消新订单 | 4: cb 双向取消]
  "price": "10300",  // 委托价格
  "timeInForce": "gtc", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
  "mini":"false", // 是否mini合约 [true: 是 | false: 否]
  "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
  "postOnly": false, // 只做maker [true: 是 | false: 否]
  "positionMerge": "long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]
  "positionId": 0,  // 提交的仓位ID
  "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
  "close": false,   // 是否平仓单 [true: 平仓 | false: 开仓]
  "leverage": 0,    // 杠杆倍数
  "action": "unknown", // 仓位行为 [unknown: 未知 | increase_long: 开多 | reduce_long: 平多 | increase_short: 开空 | reduce_short: 平空]
  "status": "accepted", // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
  "executedQty": "0", // 已成交数量
  "profit": "0",    // 收益
  "origin":0, // 强平价格标识 [当origin=-1 时代表该笔订单为强平委托]          
  "brokerId":0, // 渠道ID
  "update_id":'1125899907137993336', // 更新ID
  "executedCost": "0", // 已成交价值
  "fillCount": 0, // 成交次数
  "fills": [],
  "fees": [],
  "updateTime": "1733390650379" // 更新时间
}

```

**提交委托**

* 请求方式 POST
* 请求路径 /v1/order
* 权限: Trade
* 请求参数


| 参数名称            | 参数类型    | 是否必传 | 说明                                                                                                                                                                           |
|-----------------|---------|------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol          | string  | 是    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等                                                                                                                                  |
| side            | string  | 是    | 交易方向 [buy: 买入 | sell: 卖出]                                                                                                                                                            |
| type            | string  | 是    | 委托类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损] |
| quantity        | decimal | 是    | 委托量                                                                                                                                                                          |
| market          | string  | 是    | 市场类型 [spot: 现货 | lpc: U本位永续]                                                                                                                                                         |
| client_order_id | string  | 否    | 委托ID,有效值为int64整数的字符串,建议使用提交委托时的Unix时间戳                                                                                                                                       |
| price           | decimal | 否    | 委托限价                                                                                                                                                                         |
| positionMerge   | string  | 否    | 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]                                                                                                                                    |
| marginMethod    | string  | 否    | 保证金模式 [isolate: 逐仓 | cross: 全仓]                                                                                                                                                    |
| mini            | bool    | 否    | 是否mini合约 [true: 是 | false: 否] 当为true时必须满足 positionMerge=none&&marginMethod=isolate&&type=limit                                                                                                 |
| leverage        | int     | 否    | 杠杆倍数                                                                                                                                                                    |
| close           | bool    | 否    | 是否平仓单 [true: 平仓 | false: 开仓]                                                                                                                                                      |
| post_only       | bool    | 否    | 只做maker [true: 是 | false: 否]                                                                                                                                                                      |
| time_in_force   | string  | 否       | 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消] 默认值: gtc                                                                                      |
| positionId      | string  | 否       | 仓位ID                                                                                                                                                                         |
| trigger_price         | decimal | 否       | 触发价格    止赢止损订单使用                                                                                                                                                             |
| tpo_trigger         | int     | 否       | 开仓止盈,需配合tpo_trigger_value字段使用    0 不启用 1 启用                                                                                                                                  |
| slo_trigger         | int     | 否       | 开仓止损,需配合slo_trigger_value字段使用    0 不启用 1 启用                                                                                                                                  |
| tpo_trigger_value         | decimal | 否       | 开仓止盈触发价格                                                                                                                                                                     |
| slo_trigger_value         | decimal | 否       | 开仓止损触发价格                                                                                                                                                                     |

> 委托对象
> 最多包含该委托的20笔成交
> 如果委托有多于20笔成交,那么该对象仅包含最后20笔,其他成交请通过 fills 接口获取

## 获取订单详情

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

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
  "orderId": "4611767382287843330", // 订单ID
  "clientOrderId": "",  // 自定义订单ID
  "createTime": "1733390630904", // 创建时间
  "product": "BTC_USDT_SWAP", // 交易对代码
  "type": "limit", // 订单类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损]
  "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
  "quantity": "0.01", // 委托数量
  "stf": "disabled", // 订单成交转换方式 [0: disabled 禁用自成交保护 | 1: dc 减少并取消 | 2: co 取消老订单 | 3: cn 取消新订单 | 4: cb 双向取消]
  "price": "10300",  // 委托价格
  "timeInForce": "gtc", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
  "mini":"false", // 是否mini合约 [true: 是 | false: 否]
  "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
  "postOnly": false, // 只做maker [true: 是 | false: 否]
  "positionMerge": "long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]
  "positionId": 0,  // 提交的仓位ID
  "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
  "close": false,   // 是否平仓单 [true: 平仓 | false: 开仓]
  "leverage": 0,    // 杠杆倍数
  "action": "unknown", // 仓位行为 [unknown: 未知 | increase_long: 开多 | reduce_long: 平多 | increase_short: 开空 | reduce_short: 平空]
  "status": "filled", // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
  "executedQty": "0.01", // 已成交数量
  "profit": "0",    // 收益
  "origin":0, // 强平价格标识 [当origin=-1 时代表该笔订单为强平委托]       
  "markPrice": "10000", // 当origin=-1 代表发生强平时的标记价格
  "brokerId":0, // 渠道ID
  "update_id":'1125899907137993336', // 更新ID
  "executedCost": "103", // 已成交价值
  "fillCount": 1, // 成交次数
  "fills": [  // 成交详情
    {
      "tradeId": 1,
      "time": "1733390650379",
      "price": "10300",
      "quantity": "0.01",
      "profit": "0", // 收益
      "taker": false, // 是否为Taker [true: 是 | false: 否]
      "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
      "fees": [
        {
          "amount": "0.103", // 资产数量
          "asset": "USDT", // 资产代码
          "value": "0.103" // 估值
        }
      ]
    }
  ],
  "fees": [  // 手续费
    {
      "amount": "0.103", // 资产数量
      "asset": "USDT", // 资产代码
      "value": "0.103" // 估值
    }
  ],
  "updateTime": "1733390650379" // 更新时间
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
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'limit=2&market=spot&symbol=BTC_USDT';
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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():
    path = '/v1/history/orders'
    query_str = 'limit=2&market=spot&symbol=BTC_USDT'
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
    "orderId": "4611767382287843330", // 订单ID
    "clientOrderId": "",  // 自定义订单ID
    "createTime": "1733390630904", // 创建时间
    "product": "BTC_USDT_SWAP", // 交易对代码
    "type": "limit", // 订单类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损]
    "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
    "quantity": "0.01", // 委托数量
    "stf": "disabled", // 订单成交转换方式 [0: disabled 禁用自成交保护 | 1: dc 减少并取消 | 2: co 取消老订单 | 3: cn 取消新订单 | 4: cb 双向取消]
    "price": "10300",  // 委托价格
    "timeInForce": "gtc", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
    "mini":"false", // 是否mini合约 [true: 是 | false: 否]
    "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
    "postOnly": false, // 只做maker [true: 是 | false: 否]
    "positionMerge": "long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]
    "positionId": 0,  // 提交的仓位ID
    "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
    "close": false,   // 是否平仓单 [true: 平仓 | false: 开仓]
    "leverage": 0,    // 杠杆倍数
    "action": "unknown", // 仓位行为 [unknown: 未知 | increase_long: 开多 | reduce_long: 平多 | increase_short: 开空 | reduce_short: 平空]
    "status": "filled", // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
    "executedQty": "0.01", // 已成交数量
    "profit": "0",    // 收益
    "origin":0, // 强平价格标识 [当origin=-1 时代表该笔订单为强平委托]          
    "markPrice": "10000", // 当origin=-1 代表发生强平时的标记价格
    "brokerId":0, // 渠道ID
    "update_id":'1125899907137993336', // 更新ID
    "executedCost": "103", // 已成交价值
    "fillCount": 1, // 成交次数
    "fills": [  // 成交详情
      {
        "tradeId": 1,
        "time": "1733390650379",
        "price": "10300",
        "quantity": "0.01",
        "profit": "0", // 收益
        "taker": false, // 是否为Taker [true: 是 | false: 否]
        "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
        "fees": [
          {
            "amount": "0.103", // 资产数量
            "asset": "USDT", // 资产代码
            "value": "0.103" // 估值
          }
        ]
      }
    ],
    "fees": [  // 手续费
      {
        "amount": "0.103", // 资产数量
        "asset": "USDT", // 资产代码
        "value": "0.103" // 估值
      }
    ],
    "updateTime": "1733390650379" // 更新时间
  },
  ...
]
```

**获取ApiKey对应账户中符合下列条件的委托**

2. 三个月内的已结算委托, 含已拒绝, 已撤销和已成交委托
3. 全部已成交委托
4. 全部已撤销的部分成交委托

* 请求方式 GET
* 请求路径 /v1/history/orders
* 权限: View


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                                     |
| ------------ | ---------- |------|----------------------------------------------------------------------------------------------------------------------------------------|
| market | string   | 是    | 交易对市场 [spot: 现货 | lpc: U本位合约]<br/>                                                                                   |
| symbol     | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |
| start_time | long     | 否    | 限定返回委托的最近创建时间                                                                                                                          |
| end_time   | long     | 否    | 限定返回委托的最近创建时间                                                                                                                          |
| before     | int64    | 否    | 委托更新ID<br/>限定返回委托的最大更新ID                                                                                                              |
| after      | int64    | 否    | 委托更新ID<br/>限定返回委托的最小更新ID                                                                                                              |
| limit      | long     | 否    | 指定最多返回多少个委托                                                                                                                            |

* 该接口支持的参数组合和数据源

  *  market + symbol + start_time
  *  market + symbol + start_time + limit
  *  market + symbol + end_time
  *  market + symbol + end_time + limit
  *  market + symbol + start_time + end_time
  *  market + symbol + start_time + end_time + limit
  *  market + symbol + before
  *  market + symbol + before + limit
  *  market + symbol + after
  *  market + symbol + after + limit

> 订单委托按结算时间由早及近排序

## 获取未成交订单列表

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'market=spot&symbol=BTC_USDT';
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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():
    path = '/v1/pending/orders'
    query_str = 'market=spot&symbol=BTC_USDT'
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
    "orderId": "4611767382287843330", // 订单ID
    "clientOrderId": "",  // 自定义订单ID
    "createTime": "1733390630904", // 创建时间
    "product": "BTC_USDT_SWAP", // 交易对代码
    "type": "limit", // 订单类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损]
    "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
    "quantity": "0.01", // 委托数量
    "stf": "disabled", // 订单成交转换方式 [0: disabled 禁用自成交保护 | 1: dc 减少并取消 | 2: co 取消老订单 | 3: cn 取消新订单 | 4: cb 双向取消]
    "price": "10300",  // 委托价格
    "timeInForce": "gtc", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
    "mini":"false", // 是否mini合约 [true: 是 | false: 否]
    "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
    "postOnly": false, // 只做maker [true: 是 | false: 否]
    "positionMerge": "long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]
    "positionId": 0,  // 提交的仓位ID
    "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
    "close": false,   // 是否平仓单 [true: 平仓 | false: 开仓]
    "leverage": 0,    // 杠杆倍数
    "action": "unknown", // 仓位行为 [unknown: 未知 | increase_long: 开多 | reduce_long: 平多 | increase_short: 开空 | reduce_short: 平空]
    "status": "accepted", // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
    "executedQty": "0", // 已成交数量
    "profit": "0",    // 收益
    "origin":0, // 强平价格标识 [当origin=-1 时代表该笔订单为强平委托]          
    "brokerId":0, // 渠道ID
    "update_id":'1125899907137993336', // 更新ID
    "executedCost": "0", // 已成交价值
    "fillCount": 0, // 成交次数
    "fills": [],
    "fees": [],
    "updateTime": "1733390650379" // 更新时间
  },
  ...
]
```

**获取未成交的委托**

* 请求方式 GET
* 请求路径 /v1/pending/orders
* 权限: View


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                                     |
| ------------ | ---------- |------|----------------------------------------------------------------------------------------------------------------------------------------|
| market | string   | 是    | 交易对市场 [spot: 现货 | lpc: U本位合约]<br/>                                                                                   |
| symbol     | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |


> 返回的订单委托按创建时间由早及近排序

## 设置仓位杠杆

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
  positionId: '1125899906842649789',
  leverage:20,
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/change/leverage`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'positionId': '1125899906842649789',
        'leverage': 20
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/change/leverage'
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
   
```

**设置仓位杠杆**

* 请求方式 POST
* 请求路径 /v1/change/leverage
* 权限: Trade
* 请求参数


| 参数名称       | 参数类型   | 是否必传 | 说明      |
|------------|--------| ---------- |---------|
| positionId | string | 是       | 指定的仓位ID |
| leverage   | int    | 是   | 杠杆倍数    |

## 调整保证金

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
  positionId: '1125899906842649789',
  amount: 12,
  type: 1,
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/margin/transfer`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'positionId': '1125899906842649789',
        'amount': 12,
        'type': 1,
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/margin/transfer'
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
   
```

**调整保证金**

* 请求方式 POST
* 请求路径 /v1/margin/transfer
* 权限: Trade
* 请求参数


| 参数名称       | 参数类型   | 是否必传 | 说明             |
|------------|--------| ---------- |----------------|
| positionId | string | 是       | 指定的仓位ID |        |
| type       | int    | 是   | 调整类型 [1: 增加保证金 | 2: 减少保证金] |
| amount     | decimal | 是   | 数量             |



## 取消订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

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


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                               |
| ---------- | ---------- | ---------- |--------------------------------------------------------------------------------------------------------------------------------------------------|
| id       | string   | 是       | 委托ID<br>委托ID可以是交易所分配的,<br/>也可以是用户自定义的（在提交委托时使用client_order_id参数）。<br>当使用自定义ID时,需要在id前添加 “c:” 前缀。<br/>例如：提交委托时使用了自定义ID “123”, 在撤销委托时,需使用 “c:123”。 |
| market | string   | 是   | 交易对市场 [spot: 现货 | lpc: U本位合约]                                                                                                            |


## 取消批量订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret

const param = {
    symbol:'BTC_USDT'
}

let bodyStr = JSON.stringify(param);
const exprieTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ exprieTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/orders/delete`;

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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'

def do_request():

    param = {
        'symbol': 'BTC_USDT'
    }
    body_str = json.dumps(param)
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + body_str).encode("utf-8"), hashlib.sha256).hexdigest()
    path = '/v1/orders/delete'
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

**撤销全部委托**

* 请求方式 POST
* 请求路径 /v1/orders/delete
* 权限: Trade
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                              |
|--------| ---------- |-----|-------------------------------------------------|
| market | string   | 是   | 交易对市场 [spot: 现货 | lpc: U本位合约]           |
| symbol | string   | 是   | 交易对代码<br/>如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等 |
| side   | string   | 否   | 交易方向 [buy: 买入 | sell: 卖出]                                     |

> 如果请求被正确执行,返回空数组,否则返回错误信息

## 获取仓位

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'market=lpc&symbol=BTC_USDT_SWAP';
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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/positions'
    query_str = 'market=lpc&symbol=BTC_USDT_SWAP'
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
    "entryPrice": "109398.9", // 开仓价格
    "symbol": "BTC_USDT_SWAP", // 交易代码
    "leverage": "10.0", // 杠杆倍数
    "maintMargin": "0.0050000000", // 维持保证金率
    "side":"short", // 持仓方向 [long: 多仓 | short: 空仓]
    "quantity": "0.100", // 仓位数量 空仓0.1
    "posMargin": "1093.989", // 保证金
    "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
    "closableQty": "0.100", // 可平仓位数量
    "initMargin": "0.1000000000", // 初始保证金率
    "id": "1125899906842624158", // ID
    "orderMargin": "0",  // 委托保证金
    "mergeMode": "short"  // 仓位合并模式 [long: 合并多仓 | short: 合并空仓]
  }
  ...
]
```

**获取仓位**

* 请求方式 GET
* 请求路径 /v1/positions
* 权限: View
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                    |
| ------------ | ---------- | ---------- |-------------------------------------------------------|
| position_id   | string   | 否       | 仓位ID 如存在这个参数,优先级最高                                    |
| market | string   | 否   | 交易对市场 [spot: 现货 | lpc: U本位合约]                               |
| symbol     | string   | 否       | 配合market使用,交易对代码<br/>如 BTC_USDT_SWAP, ETH_USDT_SWAP 等 |

## 获取成交明细

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "9e03e8fda27b6e4fc6b29bb244747dcf64092996"; // your apikey
const secret = "b825a03636ca09c884ca11d71cfc4217a98cb8bf"; // your secret


const queryStr = 'limit=2&market=spot&symbol=BTC_USDT';
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
API_KEY = '9e03e8fda27b6e4fc6b29bb244747dcf64092996'
SECRET_KEY = 'b825a03636ca09c884ca11d71cfc4217a98cb8bf'


def do_request():
    path = '/v1/fills'
    query_str = 'limit=2&market=spot&symbol=BTC_USDT'
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
    "product":"BTC_USDT_SWAP", // 交易对代码
    "fees":[{"amount":"10","asset":"USDT","value":"10"}],// 手续费
    "quantity":"0.01", // 成交数量
    "orderId":"4611772879845982371", // 订单ID
    "fillId":"1125899906842624338", // 成交记录ID
    "price":"1000000", // 成交价格
    "time":"1733541360859", // 成交时间
    "taker":true, // 是否为Taker [true: 是 | false: 否]
    "side":"buy", // 交易方向 [buy: 买入 | sell: 卖出]
    "profit":"-9060", // 收益
    "tradeId":26 // 交易序号 
  },
  ...
]
```

**获取成交记录**

* 请求方式 GET
* 请求路径 /v1/fills
* 权限: View
* 请求参数(需要排序)


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                            |
| ------------ | ---------- | ---------- |-----------------------------------------------------------------------------------------------|
| market | string   | 是   | 交易对市场 [spot: 现货 | lpc: U本位合约]                                                         |
| order_id   | string   | 否       | 交易所分配的委托ID<br/>限定仅返回指定委托的成交记录<br/>如果不指定该参数,请指定 symbol                                         |
| symbol     | string   | 否       | 交易对代码<br/>如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等<br/>限定仅返回指定交易对的成交记录<br/>如果不指定该参数,请指定 order_id |
| start_time | int64    | 否       | 限定返回成交记录的最早时间                                                                                 |
| end_time   | int64    | 否       | 限定返回成交记录的最近时间                                                                                 |
| before     | int64    | 否       | 成交记录 fillId<br/>限定返回成交记录的最大id                                                                 |
| after      | int64    | 否       | 成交记录 fillId<br/>限定返回成交记录的最小id                                                                 |
| limit      | int32    | 否       | 限定返回结果的最大条数<br/>默认值 100                                                                       |

* 该接口支持的参数组合和数据源

  * symbol  --> database
  * symbol + limit  --> database
  * symbol + start_time  --> database
  * symbol + start_time + limit  --> database
  * symbol + end_time  --> database
  * symbol + end_time + limit  --> database
  * symbol + start_time + end_time  --> database
  * symbol + start_time + end_time + limit  --> database
  * symbol + before  --> database
  * symbol + before + limit  --> database
  * symbol + after  --> database
  * symbol + after + limit  --> database
  * order_id  --> database
  * order_id + limit  --> database
  * order_id + before  --> database
  * order_id + before + limit  --> database

> 返回结果按成交记录id由小到大排序

# User Data 推送

## Overview

> 例子

```javascript
const CryptoJS = require("crypto-js");
const WebSocket = require('ws');
const ktxws = 'wss://u-stream.ktx.com';
const apikey = "9e2bd17ff73e8531c0f3c26f93e48bfa402a3b13"; // your apikey
const secret = "ca55beb9e45d4f30b3959b464402319b9e12bac7"; // your secret
const sign = CryptoJS.HmacSHA256("/user/verify", secret).toString();

let wsClass = function () {
};


wsClass.prototype._initWs = async function () {
  let that = this;
  console.log(ktxws);

  let ws = new WebSocket(ktxws);
  that.ws = ws;

  ws.on('open', function open() {
    console.log(new Date(), 'open')
    ws.send(JSON.stringify({
      "method": "LOGIN",
      "auth": {
        "api-key": apikey, "api-sign": sign,
      }
    }));
    setInterval(function () {
      ws.ping(Date.now())
    },30000)
  });

  ws.on('close', data => {
    console.log('close, ', data);
  });

  ws.on('error',  data => {
    console.log('error ',data);
  });

  ws.on('ping', data => {
    console.log('ping ', data.toString('utf8'));
  });

  ws.on('pong', data => {
    console.log('pong ', data.toString('utf8'));
  });

  ws.on('message', data => {
    console.log(data.toString()) // the data may be is error message,check the data's stream is order or account
  });
};

let instance = new wsClass();

instance._initWs().catch(err => {
  console.log(err);
});

```

```python
import websocket
import hashlib
import hmac
import json
import threading
import time
from datetime import datetime

ws_url = 'wss://u-stream.ktx.com'
API_KEY = '9e2bd17ff73e8531c0f3c26f93e48bfa402a3b13'
SECRET_KEY = 'ca55beb9e45d4f30b3959b464402319b9e12bac7'
SIGN = hmac.new(SECRET_KEY.encode("utf-8"), "/user/verify".encode('utf-8'), hashlib.sha256).hexdigest()

def on_message(ws, message):
    print(message)


def on_error(ws, error):
    print(error)


def on_close(ws, close_status_code, close_msg):
    print("### closed ###")
    print("Status Code:", close_status_code)
    print("Message:", close_msg)

def ping_loop(ws):
  while True:
    time.sleep(3)

    data = {
      "ping": datetime.now().timestamp() * 1000
    }
    ws.send(json.dumps(data))


def on_open(ws):
    ws.send(json.dumps({
        "method": "LOGIN",
        "auth": {
            "api-key": API_KEY,
            "api-sign": SIGN,
        }
    }))
    print("### opened ###")
    threading.Thread(target=ping_loop, args=(ws,), daemon=True).start()



def connect():
    # websocket.enableTrace(True)

    ws = websocket.WebSocketApp(ws_url,
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever(ping_interval=0)


if __name__ == "__main__":
    connect()

```

使用 Websocket 推送服务可以及时获取账户的余额及委托变动信息。

**连接 Websocket 服务器**

请使用以下 URL 连接 Websocket 服务器：

wss://u-stream.ktx.com

**在连接时,请附加以下HTTP请求头**

* api-key
* api-sign
* api-expire-time

*具体方法请参考[Authentication](#authentication)章节*

> 数据流
> 在成功建立连接后,客户端将收到ApiKey对应账户的余额变动信息及委托变动信息。格式如下:

```json
{
  "stream": "account",
  "data": { Account }
}

{
  "stream": "order",
  "data": { Order }
}

{
  "stream": "order",
  "data": { Position }
}
```

## Account

**当账户余额发生变更时,会收到account事件**

```json
{
  "stream": "account",
  "data":  {
    "asset":"USDT",  // 资产代码
    "balance":"100",  // 总额
    "locked":"0",  // 冻结额
    "free":"100",  // 可用
    "withdrawable":"100",// 可转出额
    "collateral":false,// 是否抵扣保证金 [true: 是 | false: 否]
    "discountForMargin":"1", // 保证金抵扣率 [0: 不可用 | 0.5: 50% | 1: 100%]
    "discountForFee":"1" // 手续费抵扣率 [0: 不可用 | 0.5: 50% | 1: 100%]
  }
}
```

## Position

**当仓位信息发送变更时,会收到position事件**

```json
{
  "stream": "position",
  "data": {
      "id":"1125899906842624003", // 仓位ID
      "symbol":"BTC_USDT_SWAP", // 交易对代码
      "side":"long", // 持仓方向 [long: 多仓 | short: 空仓]
      "quantity":"0.1",   // 持仓数量
      "entryPrice":"0", // 开仓均价
      "mergeMode":"long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓]
      "marginMethod":"isolate",// 保证金模式 [isolate: 逐仓 | cross: 全仓]
      "leverage":"10.0", // 杠杆
      "initMargin":"0.1", // 起始保证金率
      "maintMargin":"0.005",// 维持保证金率
      "posMargin":"0", // 持仓保证金
      "orderMargin":"1009.8990000", // 委托保证金
      "closableQty": "0.100", // 可平仓位数量

  }
}
```
## Order

**当委托发生变更时,会收到order事件**

```json
{
  "stream": "order",
  "data":{
    "orderId": "4611767382287843330", // 订单ID
      "clientOrderId": "",  // 自定义订单ID
      "createTime": "1733390630904", // 创建时间
      "product": "BTC_USDT_SWAP", // 交易对代码
      "type": "limit", // 订单类型 [limit: 限价 | market: 市价 | take-profit: 市价止盈 | stop: 市价止损 | take-profit-limit: 限价止盈 | stop-limit: 限价止损]
      "side": "buy", // 交易方向 [buy: 买入 | sell: 卖出]
      "quantity": "0.01", // 委托数量
      "stf": "disabled", // 订单成交转换方式 [0: disabled 禁用自成交保护 | 1: dc 减少并取消 | 2: co 取消老订单 | 3: cn 取消新订单 | 4: cb 双向取消]
      "price": "10300",  // 委托价格
      "timeInForce": "gtc", // 委托时效性 [gtc: 成交前有效 | ioc: 即时剩余取消 | fok: 全数成交否则取消]
      "mini":"false", // 是否mini合约 [true: 是 | false: 否]
      "cancelAfter": 0, // N秒后自动撤销 [大于0: N秒后撤销 | 0: 永不自动撤销]
      "postOnly": false, // 只做maker [true: 是 | false: 否]
      "positionMerge": "long", // 仓位合并模式 [long: 合并多仓 | short: 合并空仓 | none: 分仓]
      "positionId": 0,  // 提交的仓位ID
      "marginMethod": "cross", // 保证金模式 [isolate: 逐仓 | cross: 全仓]
      "close": false,   // 是否平仓单 [true: 平仓 | false: 开仓]
      "leverage": 0,    // 杠杆倍数
      "action": "unknown", // 仓位行为 [unknown: 未知 | increase_long: 开多 | reduce_long: 平多 | increase_short: 开空 | reduce_short: 平空]
      "status": "filled", // 订单状态 [accepted: 已接收 | partial-filled: 部分成交挂单中 | filled: 已成交 | cancelled: 已撤销 | rejected: 已拒绝 | partially-cancelled: 部分成交已取消]
      "executedQty": "0.01", // 已成交数量
      "profit": "0",    // 收益
      "origin":0, // 强平价格标识 [当origin=-1 时代表该笔订单为强平委托]
      "markPrice": "10000", // 当origin=-1 代表发生强平时的标记价格
      "brokerId":0, // 渠道ID
      "update_id":'1125899907137993336', // 更新ID
      "executedCost": "103", // 已成交价值
      "fillCount": 1, // 成交次数
      "fills": [  // 成交详情
      {
        "tradeId": 1,
        "time": "1733390650379",
        "price": "10300",
        "quantity": "0.01",
        "profit": "0", // 收益
        "taker": false, // 是否为Taker [true: 是 | false: 否]
        "fees": [
          {
            "amount": "0.103", // 资产数量
            "asset": "USDT", // 资产代码
            "value": "0.103" // 估值
          }
        ]
      }
    ],
      "fees": [  // 手续费
      {
        "amount": "0.103", // 资产数量
        "asset": "USDT", // 资产代码
        "value": "0.103" // 估值
      }
    ],
      "updateTime": "1733390650379" // 更新时间
  }
}
```
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
const url = `${endPoint}/v1/ticker/get_all?market=forecast`
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
    path = '/v1/ticker/get_all?market=forecast'
    resp = requests.get(END_POINT + path)
    print(resp.text)
  
if __name__ == '__main__':
    do_request()
```

> Response

```json
{
      "productId": 759, // 交易对ID
      "product": "CFG_USDT", // 交易对符号 [格式: {事件代码}_{计价货币}]
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

* 请求方式 GET
* 请求路径 /v1/ticker/get_all
* 请求参数


| 参数名称 | 参数类型 | 是否必传 | 说明                                                         |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| market   | string   | 是       | 交易对市场 [forecast:预测市场]                               |

## 获取订单簿

> Request

```javascript
let request = require("request");
const endPoint = 'https://api.ktx.com/api';
const url = `${endPoint}/v1/order_book?market=forecast&symbol=2362124_FORECAST&side=sell`
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
    path = '/v1/order_book?market=forecast&symbol=2362124_FORECAST&side=sell'
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

* 请求方式 GET
* 请求路径 /v1/order_book
* 请求参数

| 参数名称    | 参数类型 | 是否必传 | 说明                                                         |
| :---------- | :------- | :------- | :----------------------------------------------------------- |
| market      | string   | 否       | 交易对市场 [forecast: 预测市场]                              |
| symbol      | string   | 是       | 预测市场交易对代码，格式为 `{marketId}_FORECAST`，如 `1897034_FORECAST`、`2362124_FORECAST` |
| level       | int32    | 否       | 最大返回深度档位。批量查询时可使用 `symbol=1897034_FORECAST,1897035_FORECAST`。有效值：1、2、5、10、20、50、100、200、500、1000，默认值 100 |
| price_scale | integer  | 否       | 价格精度合并。0=4位小数，1=3位小数，2=2位小数，3=1位小数，4=0位小数。默认值 0 |
| side        | string   | 否       | 盘口方向（buy: 买入盘(long方向)；sell: 卖出盘(short方向)），默认值 buy |

> 注：获取的订单簿默认是以 Yes(long) 方向的价格展示

