# 变更记录

## 2026-07-07 v4.2 — 新增合约汇总接口 + 全量 Java 示例

### 新增接口

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| 合约汇总数据 | GET | /v1/pu/contracts | 获取合约持仓量、资金费率、指数价格及买卖盘汇总 |

### 全量添加 Java 示例（41 个代码块）

* 使用 JDK 11+ 内置 API（`java.net.http.HttpClient` / `WebSocket`），无需第三方依赖
* 所有认证接口包含 HMAC-SHA256 签名辅助方法
* 密钥统一使用 `YOUR_API_KEY` / `YOUR_SECRET_KEY` 占位符

| 文件 | 接口数 | 类型 |
|------|--------|------|
| index.html.md | 1 | 认证示例（GET 签名） |
| _market_rest.md | 9 | 公开 GET（ping、time、coins、products、positionTier、order_book、candles、trades、ticker） |
| _market_ws.md | 1 | 行情 WebSocket 订阅 |
| _user_rest.md | 16 | 私有接口（资产、充值、提现、划转、下单、查单、撤单、杠杆、保证金、成交记录） |
| _user_ws.md | 1 | 用户 WebSocket LOGIN 认证 |
| _prediction.md | 13 | 预测市场（events、markets、ticker、order_book、下单、查单、撤单、仓位、拆分合并、成交记录） |

### 安全修复

* 修复 `_user_ws.md` 中 JavaScript 和 Python 示例的硬编码 API Key / Secret（4 处）

---

## 2026-07-06 v4.1 — 文档质量治理

### 代码示例修复（A 类 — 影响运行）

* 修复 Python `requests.get()` 缺少 `params=` 参数（_user_rest.md 7 处，_prediction.md 5 处，index.html.md 1 处）
* 修复 _market_rest.md 获取服务器时间示例 URL 和 path 错误
* 修复 _market_ws.md 中 3 处 JSON 示例缺少逗号、1 处多余 `}`、1 处尾部多余逗号
* 补充 _user_rest.md 中 2 处空 Response 示例

### 拼写 / 格式修复（B 类 — 不影响运行）

* 修复 `exprieTime` → `expireTime`（_user_rest.md 47 处，_prediction.md 22 处，index.html.md 若干处）
* 修复 `upload failed` 错误消息文案（_user_rest.md 15 处，_prediction.md 13 处，_market_rest.md 5 处，index.html.md 1 处）

### 安全修复（C 类）

* 硬编码 API Key / Secret 替换为 `YOUR_API_KEY` / `YOUR_SECRET_KEY` 占位符（_user_rest.md 34 处，_prediction.md 34 处，index.html.md 若干处）

### 描述一致性修复（D 类）

* 更新 API 描述：从"现货市场"改为"支持现货、U本位合约及预测市场"
* Error 格式改用 JSON 代码块（替代 HTML `<br/>`）
* 补充 _errors.md 缺失的错误码 -20006（CPU用量超限）和 -20007（访问频率超限）

---

## 2026-07-05 v4.0 — 文档结构模块化

### 文件拆分

将单文件 `index.html.md`（5073 行）拆分为模块化结构：

| 新文件 | 内容 | 行数 |
|--------|------|------|
| index.html.md | 基本信息、流量限制、认证 | ~200 |
| _market_rest.md | Market Data REST API | ~670 |
| _market_ws.md | Market Data WebSocket 推送 | ~495 |
| _user_rest.md | User Data REST API | ~2080 |
| _user_ws.md | User Data WebSocket 推送 | ~280 |
| _prediction.md | 预测市场 API | ~1350 |
| _errors.md | 错误码参考表 | ~50 |

### frontmatter 更新

```yaml
includes:
- market_rest
- market_ws
- user_rest
- user_ws
- prediction
- errors
- changelog
```
