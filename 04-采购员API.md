# 采购员 API

采购员接口需要 JWT 认证。基础路径：`/api/v1/procurement`

---

## 一、购物车

### 1.1 获取购物车

```
GET /api/v1/procurement/cart
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/procurement/cart" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "ID": 1,
      "UserID": 42,
      "ProductID": 1,
      "Quantity": 3,
      "Product": {
        "id": 1,
        "Name": "有机红富士苹果",
        "Price": 5000,
        "Stock": 200,
        "Images": "[\"https://oss.example.com/apple1.jpg\"]",
        "Status": "active"
      },
      "UpdatedAt": "2026-06-24T10:00:00+08:00"
    }
  ]
}
```

---

### 1.2 添加购物车项

```
POST /api/v1/procurement/cart/item
```

#### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `ProductID` | uint | 是 | 商品 ID |
| `quantity` | int | 是 | 数量（≥1） |

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/procurement/cart/item" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json" \
  -d '{"product_id": 1, "quantity": 3}'
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "ID": 1,
    "UserID": 42,
    "ProductID": 1,
    "Quantity": 3,
    "UpdatedAt": "2026-06-24T10:00:00+08:00"
  }
}
```

---

### 1.3 更新购物车项数量

```
PUT /api/v1/procurement/cart/item/:id
```

#### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `quantity` | int | 是 | 新数量（≥1） |

#### 请求示例

```bash
curl -X PUT "http://localhost:8090/api/v1/procurement/cart/item/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json" \
  -d '{"quantity": 5}'
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "ID": 1,
    "UserID": 42,
    "ProductID": 1,
    "Quantity": 5,
    "UpdatedAt": "2026-06-24T10:05:00+08:00"
  }
}
```

---

### 1.4 移除购物车项

```
DELETE /api/v1/procurement/cart/item/:id
```

#### 请求示例

```bash
curl -X DELETE "http://localhost:8090/api/v1/procurement/cart/item/1" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "Message": "已移除"
  }
}
```

---

### 1.5 清空购物车

```
DELETE /api/v1/procurement/cart
```

#### 请求示例

```bash
curl -X DELETE "http://localhost:8090/api/v1/procurement/cart" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "Message": "购物车已清空"
  }
}
```

---

## 二、采购资质

### 2.1 申请采购资质

```
POST /api/v1/procurement/qualification/apply
```

#### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `level` | int | 是 | 申请等级（1-5） |
| `LevelName` | string | 否 | 等级名称，如"初级采购员" |

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/procurement/qualification/apply" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  -H "Content-Type: application/json" \
  -d '{"level": 1, "level_name": "初级采购员"}'
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "ID": 1,
    "UserID": 42,
    "Level": 1,
    "LevelName": "初级采购员",
    "TotalAmount": 0,
    "CurrencyID": 1,
    "Status": "pending",
    "CreatedAt": "2026-06-24T10:00:00+08:00"
  }
}
```

---

### 2.2 获取我的资质

```
GET /api/v1/procurement/qualification/me
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/procurement/qualification/me" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "ID": 1,
    "UserID": 42,
    "Level": 1,
    "LevelName": "初级采购员",
    "TotalAmount": 500000,
    "CurrencyID": 1,
    "Status": "active",
    "ApprovedAt": "2026-06-22T09:00:00+08:00",
    "CreatedAt": "2026-06-22T08:00:00+08:00"
  }
}
```

#### 错误响应（无资质）

```json
{
  "head": {
    "code": "2001",
    "msg": "资源不存在"
  }
}
```

---

## 三、采购意向广场

### 3.1 浏览意向广场

```
GET /api/v1/procurement/intent/square
```

#### 查询参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `Page` | int | 否 | 1 | 页码 |
| `PageSize` | int | 否 | 10 | 每页数量 |
| `Keyword` | string | 否 | - | 关键词搜索（标题/描述） |
| `Lat` | float | 否 | - | 用户纬度（用于距离筛选） |
| `Lng` | float | 否 | - | 用户经度 |
| `RadiusKM` | float | 否 | 20 | 搜索半径（公里） |
| `MinLevel` | int | 否 | - | 最低等级要求 |

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/procurement/intent/square?keyword=水果&lat=39.9042&lng=116.4074&radius_km=20" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "ID": 5,
      "UserID": 10,
      "Title": "采购一批新鲜水果",
      "Description": "需要新鲜水果用于公司团建",
      "Products": [
        {
          "ProductName": "红富士苹果",
          "Quantity": 50,
          "QualityRequirement": "果径80mm以上"
        }
      ],
      "MinLevel": 1,
      "CommissionAmt": 2000,
      "EstimatedAmount": 150000,
      "QualityRequire": "新鲜，无腐烂",
      "DeliveryAddress": "北京市朝阳区建国路88号",
      "DeliveryName": "张三",
      "Status": "pending",
      "CreatedAt": "2026-06-24T10:00:00+08:00"
    }
  ],
  "pagination": {
    "per": 10,
    "count": 15,
    "page": 2,
    "current": 1,
    "next": 2,
    "previous": 0,
    "order": "id:desc"
  }
}
```

---

### 3.2 获取意向详情

```
GET /api/v1/procurement/intent/:id
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/procurement/intent/5" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "ID": 5,
    "Title": "采购一批新鲜水果",
    "Description": "需要新鲜水果用于公司团建",
    "Products": [
      { "ProductName": "红富士苹果", "Quantity": 50 }
    ],
    "MinLevel": 1,
    "CommissionAmt": 2000,
    "EstimatedAmount": 150000,
    "DeliveryAddress": "北京市朝阳区建国路88号",
    "DeliveryName": "张三",
    "DeliveryPhone": "13800138000",
    "LocationLat": 39.9042,
    "LocationLng": 116.4074,
    "Status": "pending",
    "CreatedAt": "2026-06-24T10:00:00+08:00"
  }
}
```

---

### 3.3 接取采购意向

```
POST /api/v1/procurement/intent/:id/accept
```

采购员接取意向，意向状态变为 `accepted`。

#### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/procurement/intent/5/accept" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "Message": "接取成功"
  }
}
```

---

### 3.4 获取意向关联的订单

```
GET /api/v1/procurement/intent/:id/order
```

#### 请求示例

```bash
curl "http://localhost:8090/api/v1/procurement/intent/5/order" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

#### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "ID": 1,
      "OrderNumber": "ORD202606241001",
      "MerchantID": 1,
      "TotalAmount": 8500,
      "Status": "pending",
      "PaymentStatus": "pending",
      "CreatedAt": "2026-06-24T11:00:00+08:00"
    }
  ]
}
```
