# User Data 推送

## 用户推送 User WS Overview

> 例子

```javascript
const CryptoJS = require("crypto-js");
const WebSocket = require('ws');
const ktxws = 'wss://u-stream.ktx.com';
const apikey = "YOUR_API_KEY"; // your apikey
const secret = "YOUR_SECRET_KEY"; // your secret
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
API_KEY = 'YOUR_API_KEY'
SECRET_KEY = 'YOUR_SECRET_KEY'
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

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.WebSocket;
import java.util.concurrent.CompletionStage;

public class KtxUserWsExample {
    static final String WS_URL = "wss://u-stream.ktx.com";
    static final String API_KEY = "YOUR_API_KEY";
    static final String SECRET_KEY = "YOUR_SECRET_KEY";

    public static void main(String[] args) throws Exception {
        String sign = hmacSha256("/user/verify", SECRET_KEY);

        WebSocket ws = HttpClient.newHttpClient().newWebSocketBuilder()
                .buildAsync(URI.create(WS_URL), new WebSocket.Listener() {
                    @Override
                    public void onOpen(WebSocket webSocket) {
                        System.out.println("connected");
                        String login = String.format(
                            "{\"method\":\"LOGIN\",\"auth\":{\"api-key\":\"%s\",\"api-sign\":\"%s\"}}",
                            API_KEY, sign);
                        webSocket.sendText(login, true);
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

    static String hmacSha256(String data, String secret) throws Exception {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(secret.getBytes(), "HmacSHA256"));
        byte[] hash = mac.doFinal(data.getBytes());
        StringBuilder hex = new StringBuilder();
        for (byte b : hash) hex.append(String.format("%02x", b));
        return hex.toString();
    }
}
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

