# User Data Endpoints



## 获取交易账户资产

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";
const expireTime = Date.now()+5000;

const queryStr = 'asset=BTC';
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/trade/accounts?${queryStr}`;

request.get(url,{
          headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time': expireTime
          },
        },

        function optionalCallback(err, httpResponse, body) {
          if (err) {
            return console.error('request failed:', err);
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
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    coin_symbol:'sUSDT'
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/depositAddr`;

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
            return console.error('request failed:', err);
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    coin_symbol:"BTC",
    amount:"0.001",
    addr:"bc1qksfjx5ezznnngk6grt04h8lwnta2mxtmjl0etm"
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/withdraw`;

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
            return console.error('request failed:', err);
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    asset:'USDT'
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/main/accounts`;

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
            return console.error('request failed:', err);
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
    "locked": "0",// 冻结额
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    symbol:'USDT',
    amount: 10,
    type: 'WALLET_TRADE',
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/transfer`;

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
            return console.error('request failed:', err);
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
| type        | string | 是    | 划转类型 [WALLET_TRADE: 钱包→交易账户；TRADE_WALLET: 交易账户→钱包] |
| transfer_id | string | 否   |自定义ID 最大长度为36 返回值会出现该字段用于幂等处理          |

## 子账户资产划转

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    'symbol':'BTC',
    'amount':'0.001',
    'sub_user_id':30000416,
    'side':'in',
    'transfer_id':'userdefineid001',
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/subaccount/transfer`;

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
            return console.error('request failed:', err);
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
| side        | string | 是    | 划转方向 [in: 主账户→子账户；out: 子账户→主账户]     |
| transfer_id | string | 否   | 自定义ID 最大长度为36 返回值会出现该字段用于幂等处理 |

## 获取账单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'asset=BTC&end_time=1651895799668&limit=10';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/ledgers?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime 
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
| type       | string   | 否       | 账单类型 [transfer: 划转；trade: 交易；fee: 手续费；rebate: 返佣；funding: 资金费用] |

* Data Source

  DB

## 创建订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

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
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/order`;

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
            return console.error('request failed:', err);
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


| 参数名称          | 参数类型 | 是否必传 | 说明                                                         |
|-----------------|---------|------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| symbol            | string   | 是       | 交易对代码，如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等          |
| side              | string   | 是       | 交易方向 [buy: 买入；sell: 卖出]                             |
| type              | string   | 是       | 委托类型 [limit: 限价；market: 市价；take-profit: 市价止盈；stop: 市价止损；take-profit-limit: 限价止盈；stop-limit: 限价止损] |
| quantity          | decimal  | 是       | 委托量                                                       |
| market            | string   | 是       | 市场类型 [spot: 现货；lpc: U本位永续]                        |
| client_order_id   | string   | 否       | 委托ID，有效值为int64整数的字符串，建议使用提交委托时的Unix时间戳 |
| price             | decimal  | 否       | 委托限价                                                     |
| positionMerge     | string   | 否       | 仓位合并模式 [long: 合并多仓；short: 合并空仓；none: 分仓]   |
| marginMethod      | string   | 否       | 保证金模式 [isolate: 逐仓；cross: 全仓]                      |
| mini              | bool     | 否       | 是否mini合约 [true: 是；false: 否]，当为true时必须满足 positionMerge=none && marginMethod=isolate && type=limit |
| leverage          | int      | 否       | 杠杆倍数                                                     |
| close             | bool     | 否       | 是否平仓单 [true: 平仓；false: 开仓]                         |
| post_only         | bool     | 否       | 只做maker [true: 是；false: 否]                              |
| time_in_force     | string   | 否       | 委托时效性 [gtc: 成交前有效；ioc: 即时剩余取消；fok: 全数成交否则取消]，默认值: gtc |
| positionId        | string   | 否       | 仓位ID                                                       |
| trigger_price     | decimal  | 否       | 触发价格，止盈止损订单使用                                   |
| tpo_trigger       | int      | 否       | 开仓止盈，需配合 tpo_trigger_value 字段使用，0: 不启用，1: 启用 |
| slo_trigger       | int      | 否       | 开仓止损，需配合 slo_trigger_value 字段使用，0: 不启用，1: 启用 |
| tpo_trigger_value | decimal  | 否       | 开仓止盈触发价格                                             |
| slo_trigger_value | decimal  | 否       | 开仓止损触发价格                                             |

> 委托对象
> 最多包含该委托的20笔成交
> 如果委托有多于20笔成交,那么该对象仅包含最后20笔,其他成交请通过 fills 接口获取

## 获取订单详情

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'id=4611772879845982339';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/order?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'limit=2&market=spot&symbol=BTC_USDT';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/history/orders?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    query_str = 'limit=2&market=spot&symbol=BTC_USDT'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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

1.三个月内的已结算委托, 含已拒绝, 已撤销和已成交委托
2.全部已成交委托
3.全部已撤销的部分成交委托

* 请求方式 GET
* 请求路径 /v1/history/orders
* 权限: View
* 请求参数


| 参数名称   | 参数类型 | 是否必传 | 说明                                                                                                                                     |
| ------------ | ---------- |------|----------------------------------------------------------------------------------------------------------------------------------------|
| market | string   | 否 | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol     | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |
| start_time | long     | 否    | 限定返回委托的最早创建时间                                                                                                                       |
| end_time   | long     | 否    | 限定返回委托的最近创建时间                                                                                              |
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'market=spot&symbol=BTC_USDT';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/pending/orders?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime  
        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    query_str = 'market=spot&symbol=BTC_USDT'
    expire_time = str(int(time.time() * 1000) + 5000)
    sign = hmac.new(SECRET_KEY.encode("utf-8"), ('' + expire_time + query_str).encode("utf-8"), hashlib.sha256).hexdigest()

    headers = {
        'Content-Type': 'application/json',
        'api-key': API_KEY,
        'api-sign': sign,
        'api-expire-time':expire_time 
    }
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
| market | string   | 是    | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol     | string   | 否    | 交易对代码,如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等<br/>当 status=unsettled 时, 不指定 symbol 将返回全部交易对的未结算委托<br/>当 status=settled 时, 必须给定 symbol 参数 |


> 返回的订单委托按创建时间由早及近排序

## 设置仓位杠杆

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
  positionId: '1125899906842649789',
  leverage:20,
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/change/leverage`;

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
            return console.error('request failed:', err);
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
{}
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
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
  positionId: '1125899906842649789',
  amount: 12,
  type: 1,
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/margin/transfer`;

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
            return console.error('request failed:', err);
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
{}
```

**调整保证金**

* 请求方式 POST
* 请求路径 /v1/margin/transfer
* 权限: Trade
* 请求参数


| 参数名称       | 参数类型   | 是否必传 | 说明             |
|------------|--------| ---------- |----------------|
| positionId | string | 是       | 指定的仓位ID |
| type       | int    | 是   | 调整类型 [1: 增加保证金；2: 减少保证金] |
| amount     | decimal | 是   | 数量             |



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
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
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
            return console.error('request failed:', err);
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


| 参数名称 | 参数类型 | 是否必传 | 说明                                                                                                                                               |
| ---------- | ---------- | ---------- |--------------------------------------------------------------------------------------------------------------------------------------------------|
| id       | string   | 是       | 委托ID<br>委托ID可以是交易所分配的,<br/>也可以是用户自定义的（在提交委托时使用client_order_id参数）。<br>当使用自定义ID时,需要在id前添加 “c:” 前缀。<br/>例如：提交委托时使用了自定义ID “123”, 在撤销委托时,需使用 “c:123”。 |
| market | string   | 是   | 交易对市场 [spot: 现货；lpc: U本位合约] |


## 取消批量订单

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const param = {
    symbol:'BTC_USDT'
}

let bodyStr = JSON.stringify(param);
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + bodyStr, secret).toString();
const url = `${endpoints}/v1/orders/delete`;

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
            return console.error('request failed:', err);
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
| market | string   | 是   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol | string   | 是   | 交易对代码<br/>如 BTC_USDT, ETH_USDT, BTC_USDT_SWAP 等 |
| side   | string   | 否   | 交易方向 [buy: 买入；sell: 卖出] |

> 如果请求被正确执行,返回空数组,否则返回错误信息

## 获取仓位

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'market=lpc&symbol=BTC_USDT_SWAP';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/positions?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime 

        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
| market | string   | 否   | 交易对市场 [spot: 现货；lpc: U本位合约] |
| symbol     | string   | 否       | 配合market使用,交易对代码<br/>如 BTC_USDT_SWAP, ETH_USDT_SWAP 等 |

## 获取成交明细

> Request

```javascript
let CryptoJS = require("crypto-js");
let request = require("request");

const endpoints = 'https://api.ktx.com/papi'
const apikey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";


const queryStr = 'limit=2&market=spot&symbol=BTC_USDT';
const expireTime = Date.now()+5000;
const sign = CryptoJS.HmacSHA256(''+ expireTime + queryStr, secret).toString();
const url = `${endpoints}/v1/fills?${queryStr}`;

request.get(url,{
        headers: {
            'Content-Type': 'application/json',
            'api-key': apikey,
            'api-sign': sign,
            'api-expire-time':expireTime 

        },
    },

    function optionalCallback(err, httpResponse, body) {
        if (err) {
            return console.error('request failed:', err);
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
    resp = requests.get(END_POINT + path, params=query_str, headers=headers)
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
| market | string   | 是   | 交易对市场 [spot: 现货；lpc: U本位合约] |
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

