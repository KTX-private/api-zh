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

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.WebSocket;
import java.util.concurrent.CompletionStage;

public class KtxMarketWsExample {
    static final String WS_URL = "wss://m-stream.ktx.com";

    public static void main(String[] args) throws Exception {
        WebSocket ws = HttpClient.newHttpClient().newWebSocketBuilder()
                .buildAsync(URI.create(WS_URL), new WebSocket.Listener() {
                    @Override
                    public void onOpen(WebSocket webSocket) {
                        System.out.println("connected");
                        String sub = "{\"method\":\"SUBSCRIBE\",\"params\":[\"spot.BTC_USDT.order_book.5\"]}";
                        webSocket.sendText(sub, true);
                        webSocket.request(1);
                    }

                    @Override
                    public CompletionStage<?> onText(WebSocket webSocket, CharSequence data, boolean last) {
                        System.out.println(data);
                        webSocket.request(1);
                        return null;
                    }

                    @Override
                    public CompletionStage<?> onClose(WebSocket webSocket, int statusCode, String reason) {
                        System.out.println("closed: " + reason);
                        return null;
                    }

                    @Override
                    public void onError(WebSocket webSocket, Throwable error) {
                        System.err.println("error: " + error.getMessage());
                    }
                }).join();

        // 保持主线程运行
        Thread.sleep(Long.MAX_VALUE);
    }
}
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
  "data": ... // 数据
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
  "method": "SUBSCRIBE",
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
  "method": "SUBSCRIBE",
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
    ]
  }
}
```

## 订阅ticker

**订阅ticker**

> 发送以下请求可订阅Ticker

```json
{
  "id": 123,
  "method": "SUBSCRIBE",
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

