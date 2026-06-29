# 支付 API

> 支付相关接口，用于创建支付订单、处理支付回调等。

**基础路径:** `/api/v1/payment`

---

## 目录

- [Alipay（支付宝）](#alipay支付宝)
- [WeChat（微信支付）](#wechat微信支付)

---

## Alipay（支付宝）

### 1. POST /deposit — 创建充值支付（支付宝/微信）

创建一笔充值订单并返回支付链接/二维码。需要用户身份认证。

**完整 URL:** `POST /api/v1/payment/alipay/deposit`

**认证:** 需要 Bearer Token

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| user_id | integer | **是** | 用户 ID |
| currency_id | integer | **是** | 币种 ID（如 1 = CNY） |
| amount | integer | **是** | 充值金额，**单位：分**（如 100 表示 1 元） |
| pay_method | string | **是** | 支付方式：`alipay` 或 `wechat` |
| return_url | string | 否 | 支付成功后同步跳转地址 |
| notify_url | string | 否 | 异步通知接收地址（不传则使用系统默认） |

**Request 示例:**

```json
{
  "UserID": 1001,
  "CurrencyID": 1,
  "Amount": 10000,
  "PayMethod": "alipay",
  "ReturnURL": "https://example.com/payment/success",
  "NotifyURL": "https://api.example.com/api/v1/payment/alipay/notify"
}
```

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "OrderID": "PAY2025012012345678",
    "PayURL": "https://pay.alipay.com/xxxxxx",
    "QRCode": "https://cdn.example.com/qrcodes/PAY2025012012345678.png",
    "Amount": 10000,
    "ExpireTime": "2025-01-20T11:00:00Z"
  }
}
```

| 响应字段 | 类型 | 说明 |
|---|---|---|
| order_id | string | 系统生成的支付订单号 |
| pay_url | string | 支付跳转链接（H5 或 PC 收银台） |
| qr_code | string | 支付二维码图片 URL（扫码支付时使用） |
| amount | integer | 实际支付金额（分） |
| expire_time | string | 支付链接过期时间 |

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/alipay/deposit' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "UserID": 1001,
    "CurrencyID": 1,
    "Amount": 10000,
    "PayMethod": "alipay",
    "ReturnURL": "https://example.com/payment/success"
  }'
```

---

### 2. POST /order — 创建订单支付

对已生成的业务订单发起支付。需要用户身份认证。

**完整 URL:** `POST /api/v1/payment/alipay/order`

**认证:** 需要 Bearer Token

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| order_id | string | **是** | 业务订单号 |
| pay_method | string | **是** | 支付方式：`alipay` 或 `wechat` |

**Request 示例:**

```json
{
  "OrderID": "ORD20250120001",
  "PayMethod": "alipay"
}
```

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "OrderID": "PAY2025012098765432",
    "PayURL": "https://pay.alipay.com/yyyyyy",
    "QRCode": "https://cdn.example.com/qrcodes/PAY2025012098765432.png",
    "Amount": 5000,
    "ExpireTime": "2025-01-20T11:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/alipay/order' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "OrderID": "ORD20250120001",
    "PayMethod": "alipay"
  }'
```

---

### 3. POST /alipay/notify — 支付宝异步通知回调

由支付宝服务器调用，通知支付结果。**不需要身份认证**（由支付宝签名验证保证安全）。

**完整 URL:** `POST /api/v1/payment/alipay/notify`

**认证:** 无（基于支付宝签名校验）

**请求头:**

| Header | 值 |
|---|---|
| Content-Type | application/x-www-form-urlencoded |

**请求体:** 支付宝异步通知以 `application/x-www-form-urlencoded` 格式发送，主要字段如下：

| 字段 | 类型 | 说明 |
|---|---|---|
| notify_time | string | 通知时间 |
| notify_type | string | 通知类型 |
| notify_id | string | 通知校验 ID |
| app_id | string | 支付宝分配给开发者的应用 ID |
| charset | string | 编码格式 |
| version | string | 接口版本 |
| sign_type | string | 签名类型（RSA2） |
| sign | string | 签名 |
| trade_no | string | 支付宝交易号 |
| out_trade_no | string | 商户订单号（即系统生成的支付订单号） |
| out_biz_no | string | 商户业务号（业务订单号） |
| buyer_id | string | 买家支付宝用户 ID |
| buyer_logon_id | string | 买家支付宝账号 |
| seller_id | string | 卖家支付宝用户 ID |
| trade_status | string | 交易状态：`TRADE_SUCCESS` 表示支付成功 |
| total_amount | string | 订单金额（单位：元） |
| receipt_amount | string | 实收金额（单位：元） |
| invoice_amount | string | 开票金额（单位：元） |
| buyer_pay_amount | string | 付款金额（单位：元） |
| point_amount | string | 集分宝金额（单位：元） |
| refund_fee | string | 总退款金额（单位：元） |
| gmt_create | string | 交易创建时间 |
| gmt_payment | string | 交易付款时间 |
| gmt_refund | string | 交易退款时间 |
| gmt_close | string | 交易关闭时间 |
| fund_bill_list | string | 支付金额信息 |

**注意:** 系统收到通知后需验证签名并处理业务逻辑（更新订单状态）。处理完成后需返回 `success`（纯文本，无 JSON）给支付宝。若返回其他内容，支付宝会定期重发通知。

**系统处理成功后响应 (HTTP 200):**

```
success
```

**系统处理失败时响应 (HTTP 200):**

```
fail
```

**Request 示例 (application/x-www-form-urlencoded):**

```
notify_time=2025-01-20 10:30:00&notify_type=trade_status_sync&notify_id=2025012000123456&app_id=2021001000000000&charset=utf-8&version=1.0&sign_type=RSA2&sign=xxxxxx&trade_no=2025012022000000000000001&out_trade_no=PAY2025012012345678&out_biz_no=ORD20250120001&buyer_id=2088000000000001&buyer_logon_id=test@example.com&seller_id=2088000000000002&trade_status=TRADE_SUCCESS&total_amount=100.00&receipt_amount=100.00&buyer_pay_amount=100.00&gmt_create=2025-01-20 10:29:00&gmt_payment=2025-01-20 10:30:00&fund_bill_list=[{"Amount":"100.00","fundChannel":"ALIPAYACCOUNT"}]
```

**Curl 示例（模拟支付宝通知）:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/alipay/notify' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'notify_time=2025-01-20+10%3A30%3A00&notify_type=trade_status_sync&notify_id=2025012000123456&app_id=2021001000000000&charset=utf-8&version=1.0&sign_type=RSA2&sign=xxxxxx&trade_no=2025012022000000000000001&out_trade_no=PAY2025012012345678&trade_status=TRADE_SUCCESS&total_amount=100.00&receipt_amount=100.00&buyer_pay_amount=100.00&gmt_create=2025-01-20+10%3A29%3A00&gmt_payment=2025-01-20+10%3A30%3A00'
```

---

### 4. GET /alipay/return — 支付宝同步返回 URL

用户在支付宝支付完成后，被重定向回此地址。**仅做页面展示，不做订单状态更新依据**（以异步通知为准）。

**完整 URL:** `GET /api/v1/payment/alipay/return`

**认证:** 无

**Query 参数由支付宝附加:**

| 参数 | 类型 | 说明 |
|---|---|---|
| app_id | string | 支付宝应用 ID |
| out_trade_no | string | 商户订单号 |
| trade_no | string | 支付宝交易号 |
| total_amount | string | 订单金额 |
| sign | string | 签名 |
| sign_type | string | 签名类型 |
| charset | string | 编码 |
| method | string | 接口名称 |
| timestamp | string | 时间戳 |
| version | string | 版本 |

**行为:** 系统应验证签名，然后重定向用户到前端支付成功页面（如 `return_url` 所示），或展示支付结果页面。

**重定向行为示例:**

```
HTTP/1.1 302 Found
Location: https://example.com/payment/success?order_id=PAY2025012012345678&trade_no=2025012022000000000000001
```

或展示一个简单的支付结果页面（HTML）。

**Curl 示例（模拟用户支付后返回）:**

```bash
curl -X GET 'https://api.example.com/api/v1/payment/alipay/return?app_id=2021001000000000&out_trade_no=PAY2025012012345678&trade_no=2025012022000000000000001&total_amount=100.00&sign=xxxxxx&sign_type=RSA2&charset=utf-8&method=alipay.trade.page.pay.return&timestamp=2025-01-20+10%3A30%3A00&version=1.0'
```

---

## WeChat（微信支付）

### 5. POST /wechat/deposit — 创建微信充值支付

**完整 URL:** `POST /api/v1/payment/wechat/deposit`

**认证:** 需要 Bearer Token

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| user_id | integer | **是** | 用户 ID |
| currency_id | integer | **是** | 币种 ID |
| amount | integer | **是** | 充值金额，**单位：分** |
| return_url | string | 否 | 支付成功后的跳转地址 |
| notify_url | string | 否 | 异步通知地址 |

**Request 示例:**

```json
{
  "UserID": 1001,
  "CurrencyID": 1,
  "Amount": 5000,
  "ReturnURL": "https://example.com/payment/success",
  "NotifyURL": "https://api.example.com/api/v1/payment/wechat/notify"
}
```

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "OrderID": "PAY2025012055555555",
    "PayURL": "https://wx.tenpay.com/xxxxxx",
    "QRCode": "https://cdn.example.com/qrcodes/PAY2025012055555555.png",
    "Amount": 5000,
    "ExpireTime": "2025-01-20T11:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/wechat/deposit' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "UserID": 1001,
    "CurrencyID": 1,
    "Amount": 5000,
    "ReturnURL": "https://example.com/payment/success"
  }'
```

---

### 6. POST /wechat/order — 创建微信订单支付

**完整 URL:** `POST /api/v1/payment/wechat/order`

**认证:** 需要 Bearer Token

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| order_id | string | **是** | 业务订单号 |

**Request 示例:**

```json
{
  "OrderID": "ORD20250120002"
}
```

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "OrderID": "PAY2025012066666666",
    "PayURL": "https://wx.tenpay.com/yyyyyy",
    "QRCode": "https://cdn.example.com/qrcodes/PAY2025012066666666.png",
    "Amount": 8800,
    "ExpireTime": "2025-01-20T11:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/wechat/order' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "OrderID": "ORD20250120002"
  }'
```

---

### 7. POST /wechat/notify — 微信支付异步通知回调

由微信支付服务器调用，通知支付结果。**不需要身份认证**（由微信签名验证保证安全）。

**完整 URL:** `POST /api/v1/payment/wechat/notify`

**认证:** 无（基于微信签名校验）

**请求头:**

| Header | 值 |
|---|---|
| Content-Type | application/xml |

**请求体 (XML):** 微信通知以 XML 格式发送，并经过加密。以下是解密后的主要字段：

| XML 字段 | 类型 | 说明 |
|---|---|---|
| return_code | string | 返回状态码：`SUCCESS` / `FAIL` |
| return_msg | string | 返回信息 |
| appid | string | 微信分配的公众账号 ID |
| mch_id | string | 微信支付商户号 |
| device_info | string | 设备号 |
| nonce_str | string | 随机字符串 |
| sign | string | 签名 |
| result_code | string | 业务结果：`SUCCESS` / `FAIL` |
| err_code | string | 错误代码 |
| err_code_des | string | 错误代码描述 |
| openid | string | 用户在商户应用下的用户标识 |
| is_subscribe | string | 是否关注公众账号 |
| trade_type | string | 交易类型：JSAPI、NATIVE、APP 等 |
| bank_type | string | 付款银行类型 |
| total_fee | integer | 订单金额，**单位：分** |
| settlement_total_fee | integer | 应结订单金额（分） |
| fee_type | string | 货币类型（CNY） |
| cash_fee | integer | 现金支付金额（分） |
| cash_fee_type | string | 现金支付货币类型 |
| coupon_fee | integer | 代金券或立减优惠金额 |
| coupon_count | integer | 代金券使用数量 |
| transaction_id | string | 微信支付订单号 |
| out_trade_no | string | 商户订单号（即系统生成的支付订单号） |
| attach | string | 附加数据 |
| time_end | string | 支付完成时间（格式 `yyyyMMddHHmmss`） |

**Request 示例 (XML):**

```xml
<xml>
  <return_code><![CDATA[SUCCESS]]></return_code>
  <appid><![CDATA[wx1234567890abcdef]]></appid>
  <mch_id><![CDATA[1234567890]]></mch_id>
  <nonce_str><![CDATA[abcdef123456]]></nonce_str>
  <sign><![CDATA[XXXXXXSIGNXXXXXX]]></sign>
  <result_code><![CDATA[SUCCESS]]></result_code>
  <openid><![CDATA[oUpF8uMEb4qRXfQ1StZqk="]]></openid>
  <is_subscribe><![CDATA[Y]]></is_subscribe>
  <trade_type><![CDATA[NATIVE]]></trade_type>
  <bank_type><![CDATA[CMC]]></bank_type>
  <total_fee>5000</total_fee>
  <settlement_total_fee>5000</settlement_total_fee>
  <fee_type><![CDATA[CNY]]></fee_type>
  <cash_fee>5000</cash_fee>
  <cash_fee_type><![CDATA[CNY]]></cash_fee_type>
  <transaction_id><![CDATA[2025012020000000000000000001]]></transaction_id>
  <out_trade_no><![CDATA[PAY2025012055555555]]></out_trade_no>
  <time_end><![CDATA[20250120103000]]></time_end>
</xml>
```

**系统处理成功后响应 (HTTP 200, XML):**

```xml
<xml>
  <return_code><![CDATA[SUCCESS]]></return_code>
  <return_msg><![CDATA[OK]]></return_msg>
</xml>
```

**系统处理失败时响应 (HTTP 200, XML):**

```xml
<xml>
  <return_code><![CDATA[FAIL]]></return_code>
  <return_msg><![CDATA[签名失败]]></return_msg>
</xml>
```

**Curl 示例（模拟微信通知）:**

```bash
curl -X POST 'https://api.example.com/api/v1/payment/wechat/notify' \
  -H 'Content-Type: application/xml' \
  -d '<xml>
    <return_code><![CDATA[SUCCESS]]></return_code>
    <appid><![CDATA[wx1234567890abcdef]]></appid>
    <mch_id><![CDATA[1234567890]]></mch_id>
    <nonce_str><![CDATA[abcdef123456]]></nonce_str>
    <sign><![CDATA[XXXXXXSIGNXXXXXX]]></sign>
    <result_code><![CDATA[SUCCESS]]></result_code>
    <openid><![CDATA[oUpF8uMEb4qRXfQ1StZqk=]]></openid>
    <is_subscribe><![CDATA[Y]]></is_subscribe>
    <trade_type><![CDATA[NATIVE]]></trade_type>
    <total_fee>5000</total_fee>
    <transaction_id><![CDATA[2025012020000000000000000001]]></transaction_id>
    <out_trade_no><![CDATA[PAY2025012055555555]]></out_trade_no>
    <time_end><![CDATA[20250120103000]]></time_end>
  </xml>'
```

---

### 8. GET /wechat/return — 微信同步返回 URL

用户在微信内支付完成后，被重定向回此地址。**仅做页面展示，不做订单状态更新依据**（以异步通知为准）。

**完整 URL:** `GET /api/v1/payment/wechat/return`

**认证:** 无

**行为:** 系统根据支付结果查询订单状态，然后重定向用户到前端支付成功/失败页面。

**重定向行为示例:**

```
HTTP/1.1 302 Found
Location: https://example.com/payment/success?order_id=PAY2025012055555555
```

或展示一个简单的支付结果页面（HTML）。

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/payment/wechat/return?order_id=PAY2025012055555555'
```

---

## 通用错误码

| HTTP 状态码 | code | message | 说明 |
|---|---|---|---|
| 400 | 40000 | Bad Request | 请求参数错误 |
| 401 | 40100 | Unauthorized | 未认证或 token 过期 |
| 402 | 40200 | Payment Required | 支付失败（余额不足等） |
| 404 | 40400 | Not Found | 订单不存在 |
| 409 | 40900 | Conflict | 订单状态冲突（已支付） |
| 422 | 42200 | Validation Error | 请求参数校验失败 |
| 500 | 50000 | Internal Server Error | 服务器内部错误 |

**错误响应示例:**

```json
{
  "head": {
    "code": "1001",
    "msg": "金额必须大于 0"
  }
}
```
