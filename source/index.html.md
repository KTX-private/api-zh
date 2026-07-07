---
title: Ktx API文档(v4)

language_tabs:
- javascript
- python
- java

toc_footers:
- <a href='https://www.ktx.com'>开始交易</a>

includes:
- market_rest
- market_ws
- user_rest
- user_ws
- prediction
- errors

search: true

code_clipboard: true

---
# 基本信息

## API

**使用 API 开发应用程序，您可以获取 Ktx 市场的行情数据，进行自动化交易。支持现货、U本位合约及预测市场等多种交易类型。API 包含众多接口，按功能分为以下几组：**

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

```json
{
  "state": -10001,
  "msg": "error message"
}
```

*其中，state 表示错误码，msg 包含错误原因或修复建议。具体错误类型请参考 [Error](#errors) 章节。*

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
const apiKey = "YOUR_API_KEY";
const secret = "YOUR_SECRET_KEY";

const queryStr = 'asset=BTC';
const expireTime = Date.now() + 5000;
const sign = CryptoJS.HmacSHA256('' + expireTime + queryStr, secret).toString(); // POST 请求请用 bodyStr 替换 queryStr
const url = `${endpoints}/v1/trade/accounts?${queryStr}`;

request.get(url, {
    headers: {
        'Content-Type': 'application/json',
        'api-key': apiKey,
        'api-sign': sign,
        'api-expire-time': expireTime
    },
}, function (err, httpResponse, body) {
    if (err) {
        return console.error('request failed:', err);
    }
    console.log(body);
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
    # POST 请求请用 body_str 替换 query_str
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

```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class KtxApiExample {
    static final String ENDPOINT = "https://api.ktx.com/papi";
    static final String API_KEY = "YOUR_API_KEY";
    static final String SECRET_KEY = "YOUR_SECRET_KEY";

    public static void main(String[] args) throws Exception {
        String path = "/v1/trade/accounts";
        String queryStr = "asset=BTC";
        long expireTime = System.currentTimeMillis() + 5000;
        // POST 请求请用 bodyStr 替换 queryStr
        String sign = hmacSha256("" + expireTime + queryStr, SECRET_KEY);

        URI uri = URI.create(ENDPOINT + path + "?" + queryStr);
        HttpRequest request = HttpRequest.newBuilder(uri)
                .header("Content-Type", "application/json")
                .header("api-key", API_KEY)
                .header("api-sign", sign)
                .header("api-expire-time", String.valueOf(expireTime))
                .GET()
                .build();

        HttpResponse<String> response = HttpClient.newHttpClient()
                .send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(response.body());
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
