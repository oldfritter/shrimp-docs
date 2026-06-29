# 客户前端 API

客户前端接口需要 JWT 认证，请求头携带 `Authorization: Bearer <token>`。

基础路径：`/api/v1/customer`

---

## 一、订单管理

### 1.1 获取订单列表

```
GET /api/v1/customer/order
```

#### 查询参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `Page` | int | 否 | 1 | 页码 |
| `PageSize` | int | 否 | 10 | 每页数量 |
| `status` | string | 否 | - | 订单状态过滤 |

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/order?page=1&page_size=10" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": [
    {
      "ID": 1,
      "OrderNumber": "ORD202606241001",
      "Status": "pending",
      "TotalAmount": 8500,
      "PaymentStatus": "pending",
      "ShippingAddress": "北京市朝阳区建国路88号",
      "OrderItems": [
        {
          "ID": 1,
          "ProductName": "有机红富士苹果",
          "Quantity": 2,
          "UnitPrice": 2500,
          "Subtotal": 5000
        }
      ],
      "CreatedAt": "2026-06-24T10:00:00+08:00",
      "UpdatedAt": "2026-06-24T10:00:00+08:00"
    }
  ],
  "Pagination": {
    "Per": 10,
    "Count": 12,
    "Page": 2,
    "Current": 1,
    "Next": 2,
    "Previous": 0,
    "Order": "id:desc"
  }
}
```

---

### 1.2 获取订单详情

```
GET /api/v1/customer/order/:id
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/order/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "ID": 1,
    "OrderNumber": "ORD202606241001",
    "Status": "picked",
    "TotalAmount": 8500,
    "PaymentStatus": "pending",
    "ShippingAddress": "北京市朝阳区建国路88号",
    "OrderItems": [
      {
        "ID": 1,
        "ProductName": "有机红富士苹果",
        "Quantity": 2,
        "UnitPrice": 2500,
        "Subtotal": 5000
      },
      {
        "ID": 2,
        "ProductName": "精选阿克苏冰糖心",
        "Quantity": 1,
        "UnitPrice": 3500,
        "Subtotal": 3500
      }
    ],
    "CreatedAt": "2026-06-24T10:00:00+08:00",
    "UpdatedAt": "2026-06-24T14:30:00+08:00"
  }
}
```

---

### 1.3 取消订单

```
POST /api/v1/customer/order/:id/cancel
```

仅 `pending`（待确认）状态的订单可以取消。

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/order/1/cancel" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "Message": "取消成功"
  }
}
```

---

### 1.4 确认收货

```
POST /api/v1/customer/order/:id/confirm
```

仅 `picked`（已取货）状态的订单可以确认。确认后订单状态变为 `delivered`，支付状态变为 `pending_settlement`（3 天后自动结清）。

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/order/1/confirm" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "Message": "确认收货成功"
  }
}
```

---

### 1.5 创建退款申请

```
POST /api/v1/customer/order/refund
```

#### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `OrderID` | uint | 是 | 订单 ID |
| `amount` | int64 | 是 | 退款金额，单位：分 |
| `reason` | string | 是 | 退款原因 |

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/order/refund" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json" \
  -d '{
    "OrderID": 1,
    "amount": 8500,
    "Reason": "商品品质不符"
  }'
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "ID": 1,
    "OrderID": 1,
    "amount": 8500,
    "Reason": "商品品质不符",
    "Status": "pending",
    "CreatedAt": "2026-06-24T15:00:00+08:00"
  }
}
```

---

### 1.6 退款列表

```
GET /api/v1/customer/order/refund
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/order/refund" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": [
    {
      "ID": 1,
      "IncomeID": 0,
      "OrderID": 1,
      "amount": 8500,
      "Reason": "商品品质不符",
      "Status": "pending",
      "CreatedAt": "2026-06-24T15:00:00+08:00"
    }
  ]
}
```

---

## 二、物流追踪

### 2.1 获取物流信息

```
GET /api/v1/customer/shipping/track/:id
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/shipping/track/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "OrderID": 1,
    "TrackingNumber": "SF1234567890",
    "Carrier": "顺丰速运",
    "Status": "in_transit",
    "events": [
      {
        "time": "2026-06-24T12:00:00+08:00",
        "location": "北京中转站",
        "description": "快件已到达北京中转站"
      },
      {
        "time": "2026-06-24T10:00:00+08:00",
        "location": "深圳仓库",
        "description": "已揽收"
      }
    ]
  }
}
```

---

## 三、消息通知

### 3.1 获取消息列表

```
GET /api/v1/customer/message
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/message" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": [
    {
      "ID": 1,
      "Title": "采购订单已确认",
      "Content": "您的采购意向 #5 已被采购员接取",
      "IsRead": false,
      "CreatedAt": "2026-06-24T11:00:00+08:00"
    }
  ]
}
```

---

### 3.2 标记消息为已读

```
POST /api/v1/customer/message/:id/read
```

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/message/1/read" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "Message": "已标记为已读"
  }
}
```

---

### 3.3 删除消息

```
DELETE /api/v1/customer/message/:id
```

#### 请求示例

```bash
curl -X DELETE "http://localhost:8090/api/v1/customer/message/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "Message": "已删除"
  }
}
```

---

## 四、采购意向

### 4.1 创建采购意向

```
POST /api/v1/customer/intent
```

#### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `title` | string | 是 | 采购标题 |
| `description` | string | 否 | 详细描述 |
| `products` | array | 是 | 需要采购的货品清单 |
| `products[].product_name` | string | 是 | 商品名称 |
| `products[].quantity` | int | 是 | 数量 |
| `products[].quality_requirement` | string | 否 | 品质要求 |
| `products[].price_range_min` | int64 | 否 | 单价区间（最小），单位：分 |
| `products[].price_range_max` | int64 | 否 | 单价区间（最大），单位：分 |
| `MinLevel` | int | 是 | 最低采购员等级要求（1-5） |
| `CommissionAmt` | int64 | 是 | 采购员佣金，单位：分 |
| `CommissionCurrencyID` | uint | 是 | 佣金币种 ID（1=人民币） |
| `EstimatedAmount` | int64 | 否 | 预估总金额，单位：分 |
| `QualityRequire` | string | 否 | 品质要求（全局） |
| `LocationLat` | float | 是 | 纬度 |
| `LocationLng` | float | 是 | 经度 |
| `LocationName` | string | 否 | 位置名称 |
| `DeliveryAddress` | string | 是 | 收货地址 |
| `DeliveryName` | string | 是 | 收货人姓名 |
| `DeliveryPhone` | string | 是 | 收货人电话 |

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/intent" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json" \
  -d '{
    "Title": "采购一批新鲜水果",
    "description": "需要新鲜水果用于公司团建",
    "Products": [
      {
        "ProductName": "红富士苹果",
        "Quantity": 50,
        "QualityRequirement": "果径80mm以上，无虫眼",
        "PriceRangeMin": 3000,
        "PriceRangeMax": 5000
      },
      {
        "ProductName": "进口香蕉",
        "Quantity": 30,
        "QualityRequirement": "成熟度适中，无黑斑",
        "PriceRangeMin": 2000,
        "PriceRangeMax": 4000
      }
    ],
    "MinLevel": 1,
    "CommissionAmt": 2000,
    "CommissionCurrencyID": 1,
    "EstimatedAmount": 150000,
    "QualityRequire": "新鲜，无腐烂，当天采摘",
    "LocationLat": 39.9042,
    "LocationLng": 116.4074,
    "LocationName": "北京市朝阳区",
    "DeliveryAddress": "北京市朝阳区建国路88号",
    "DeliveryName": "张三",
    "DeliveryPhone": "13800138000"
  }'
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "ID": 5,
    "UserID": 42,
    "UserName": "zhangsan",
    "Title": "采购一批新鲜水果",
    "description": "需要新鲜水果用于公司团建",
    "Products": [
      {
        "ProductName": "红富士苹果",
        "Quantity": 50,
        "QualityRequirement": "果径80mm以上，无虫眼",
        "PriceRangeMin": 3000,
        "PriceRangeMax": 5000
      }
    ],
    "MinLevel": 1,
    "CommissionAmt": 2000,
    "CommissionCurrencyID": 1,
    "CommissionCurrency": "人民币",
    "EstimatedAmount": 150000,
    "QualityRequire": "新鲜，无腐烂，当天采摘",
    "LocationLat": 39.9042,
    "LocationLng": 116.4074,
    "LocationName": "北京市朝阳区",
    "DeliveryAddress": "北京市朝阳区建国路88号",
    "DeliveryName": "张三",
    "DeliveryPhone": "13800138000",
    "Status": "pending",
    "CreatedAt": "2026-06-24T10:00:00+08:00"
  }
}
```

---

### 4.2 获取我的采购意向列表

```
GET /api/v1/customer/intent/me
```

#### 查询参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `status` | string | 否 | - | 状态过滤：`pending` / `accepted` / `completed` / `cancelled` |
| `Page` | int | 否 | 1 | 页码 |
| `PageSize` | int | 否 | 10 | 每页数量 |

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/customer/intent/me?status=pending" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": [
    {
      "ID": 5,
      "Title": "采购一批新鲜水果",
      "Status": "pending",
      "CommissionAmt": 2000,
      "EstimatedAmount": 150000,
      "Products": [
        { "ProductName": "红富士苹果", "Quantity": 50 }
      ],
      "DeliveryAddress": "北京市朝阳区建国路88号",
      "CreatedAt": "2026-06-24T10:00:00+08:00",
      "ProcurementName": null
    }
  ],
  "Pagination": {
    "Per": 10,
    "Count": 3,
    "Page": 1,
    "Current": 1,
    "Next": 0,
    "Previous": 0,
    "Order": "id:desc"
  }
}
```

---

### 4.3 取消采购意向

```
POST /api/v1/customer/intent/:id/cancel
```

仅 `pending` 状态的意向可以取消。

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/customer/intent/5/cancel" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
    "Message": "取消成功"
  }
}
```
