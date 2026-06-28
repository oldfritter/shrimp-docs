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
      "id": 1,
      "user_id": 42,
      "product_id": 1,
      "quantity": 3,
      "product": {
        "id": 1,
        "name": "有机红富士苹果",
        "price": 5000,
        "stock": 200,
        "images": "[\"https://oss.example.com/apple1.jpg\"]",
        "status": "active"
      },
      "updated_at": "2026-06-24T10:00:00+08:00"
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
| `product_id` | uint | 是 | 商品 ID |
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
    "id": 1,
    "user_id": 42,
    "product_id": 1,
    "quantity": 3,
    "updated_at": "2026-06-24T10:00:00+08:00"
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
    "id": 1,
    "user_id": 42,
    "product_id": 1,
    "quantity": 5,
    "updated_at": "2026-06-24T10:05:00+08:00"
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
    "message": "已移除"
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
    "message": "购物车已清空"
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
| `level_name` | string | 否 | 等级名称，如"初级采购员" |

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
    "id": 1,
    "user_id": 42,
    "level": 1,
    "level_name": "初级采购员",
    "total_amount": 0,
    "currency_id": 1,
    "currency_name": "人民币",
    "status": "pending",
    "created_at": "2026-06-24T10:00:00+08:00"
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
    "id": 1,
    "user_id": 42,
    "level": 1,
    "level_name": "初级采购员",
    "total_amount": 500000,
    "currency_id": 1,
    "currency_name": "人民币",
    "status": "active",
    "approved_at": "2026-06-22T09:00:00+08:00",
    "created_at": "2026-06-22T08:00:00+08:00"
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
| `page` | int | 否 | 1 | 页码 |
| `page_size` | int | 否 | 10 | 每页数量 |
| `keyword` | string | 否 | - | 关键词搜索（标题/描述） |
| `lat` | float | 否 | - | 用户纬度（用于距离筛选） |
| `lng` | float | 否 | - | 用户经度 |
| `radius_km` | float | 否 | 20 | 搜索半径（公里） |
| `min_level` | int | 否 | - | 最低等级要求 |

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
      "id": 5,
      "user_id": 10,
      "user_name": "zhangsan",
      "title": "采购一批新鲜水果",
      "description": "需要新鲜水果用于公司团建",
      "products": [
        {
          "product_name": "红富士苹果",
          "quantity": 50,
          "quality_requirement": "果径80mm以上",
          "price_range_min": 3000,
          "price_range_max": 5000
        }
      ],
      "min_level": 1,
      "commission_amt": 2000,
      "commission_currency_id": 1,
      "commission_currency": "人民币",
      "estimated_amount": 150000,
      "quality_require": "新鲜，无腐烂",
      "location_lat": 39.9042,
      "location_lng": 116.4074,
      "location_name": "北京市朝阳区",
      "delivery_address": "北京市朝阳区建国路88号",
      "delivery_name": "张三",
      "status": "pending",
      "created_at": "2026-06-24T10:00:00+08:00"
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
    "id": 5,
    "title": "采购一批新鲜水果",
    "description": "需要新鲜水果用于公司团建",
    "products": [
      { "product_name": "红富士苹果", "quantity": 50 }
    ],
    "min_level": 1,
    "commission_amt": 2000,
    "estimated_amount": 150000,
    "delivery_address": "北京市朝阳区建国路88号",
    "delivery_name": "张三",
    "delivery_phone": "13800138000",
    "location_lat": 39.9042,
    "location_lng": 116.4074,
    "status": "pending",
    "user_name": "zhangsan",
    "created_at": "2026-06-24T10:00:00+08:00"
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
    "message": "接取成功"
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
      "id": 1,
      "order_number": "ORD202606241001",
      "merchant_id": 1,
      "merchant_name": "鲜果超市旗舰店",
      "total_amount": 8500,
      "status": "pending",
      "payment_status": "pending",
      "created_at": "2026-06-24T11:00:00+08:00"
    }
  ]
}
```
