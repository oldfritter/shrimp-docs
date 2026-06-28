# 公共 API

公共接口无需认证即可访问，包括商品浏览、分类查询、用户注册和登录。

---

## 1. 获取商品列表

```
GET /api/v1/product
```

### 查询参数

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `page` | int | 否 | 1 | 页码 |
| `page_size` | int | 否 | 10 | 每页数量 |
| `category_id` | uint | 否 | - | 分类 ID 筛选 |
| `status` | string | 否 | - | 状态过滤，如 `active` |
| `sort` | string | 否 | `id:desc` | 排序，支持 `id:asc`、`price:asc` 等 |

### 请求示例

```bash
curl "http://localhost:8090/api/v1/product?page=1&page_size=10&category_id=1"
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "merchant_id": 1,
      "name": "新鲜有机苹果",
      "description": "产地直供，新鲜有机苹果，每箱约5斤",
      "price": 5000,
      "stock": 200,
      "category_id": 1,
      "brand": "果园直供",
      "sku": "APL-2024-001",
      "images": "[\"https://oss.example.com/apple1.jpg\",\"https://oss.example.com/apple2.jpg\"]",
      "status": "active",
      "created_at": "2026-06-24T10:00:00+08:00",
      "updated_at": "2026-06-24T10:00:00+08:00"
    }
  ],
  "pagination": {
    "per": 10,
    "count": 56,
    "page": 6,
    "current": 1,
    "next": 2,
    "previous": 0,
    "order": "id:desc"
  }
}
```

---

## 2. 获取商品详情

```
GET /api/v1/product/:id
```

### 路径参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | uint | 商品 ID |

### 请求示例

```bash
curl "http://localhost:8090/api/v1/product/1"
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "merchant_id": 1,
    "name": "新鲜有机苹果",
    "description": "产地直供，新鲜有机苹果，每箱约5斤",
    "price": 5000,
    "currency_id": 1,
    "stock": 200,
    "category_id": 1,
    "brand": "果园直供",
    "sku": "APL-2024-001",
    "images": "[\"https://oss.example.com/apple1.jpg\"]",
    "status": "active",
    "created_at": "2026-06-24T10:00:00+08:00",
    "updated_at": "2026-06-24T10:00:00+08:00",
    "category": {
      "id": 1,
      "name": "水果",
      "description": "新鲜水果",
      "parent_id": null,
      "status": "active"
    },
    "merchant": {
      "id": 1,
      "name": "鲜果超市旗舰店",
      "code": "FRESH001",
      "status": "active"
    }
  }
}
```

### 错误响应

```json
{
  "head": {
    "code": "2001",
    "msg": "商品不存在"
  }
}
```

---

## 3. 获取分类列表

```
GET /api/v1/category
```

### 请求示例

```bash
curl "http://localhost:8090/api/v1/category"
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "name": "水果",
      "description": "新鲜水果",
      "parent_id": null,
      "status": "active"
    },
    {
      "id": 2,
      "name": "苹果",
      "description": "各种苹果",
      "parent_id": 1,
      "status": "active"
    }
  ]
}
```

---

## 4. 获取分类详情

```
GET /api/v1/category/:id
```

### 请求示例

```bash
curl "http://localhost:8090/api/v1/category/1"
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "水果",
    "description": "新鲜水果",
    "parent_id": null,
    "status": "active",
    "children": [
      {
        "id": 2,
        "name": "苹果",
        "parent_id": 1
      },
      {
        "id": 3,
        "name": "香蕉",
        "parent_id": 1
      }
    ]
  }
}
```

---

## 5. 用户注册

```
POST /api/v1/register
```

### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `username` | string | 是 | 用户名，3-50 字符 |
| `phone` | string | 是 | 手机号（唯一标识） |
| `password` | string | 是 | 密码，至少 6 位含大小写字母和数字 |

### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/register" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "zhangsan",
    "phone": "13800138000",
    "password": "Pass1234"
  }'
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "message": "注册成功",
    "user_id": 42,
    "username": "zhangsan"
  }
}
```

### 错误响应（用户名或手机号已存在）

```json
{
  "head": {
    "code": "1007",
    "msg": "用户名或手机号已被注册"
  }
}
```

---

## 6. 用户登录

```
POST /api/v1/login
```

### 请求体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `username` | string | 是 | 用户名 |
| `password` | string | 是 | 密码 |

### 请求示例

```bash
curl -X POST "http://localhost:8090/api/v1/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "zhangsan",
    "password": "Pass1234"
  }'
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0MiwiZXhwIjoxNzE5MjM0NTY3fQ...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo0MiwiZXhwIjoxNzE5NDA3MzY3fQ...",
    "token_type": "access",
    "expires_in": 7200,
    "user_id": 42,
    "username": "zhangsan",
    "phone": "13800138000",
    "roles": [
      {
        "id": 1,
        "name": "普通用户",
        "admin": false
      }
    ]
  }
}
```

### 错误响应

```json
{
  "head": {
    "code": "1401",
    "msg": "用户名或密码错误"
  }
}
```

---

## 7. 获取个人资料

```
GET /api/v1/profile
```

### 请求头

```
Authorization: Bearer <access_token>
```

### 请求示例

```bash
curl "http://localhost:8090/api/v1/profile" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 42,
    "username": "zhangsan",
    "phone": "13800138000",
    "avatar": "",
    "status": "active",
    "roles": [
      {
        "id": 1,
        "name": "普通用户",
        "admin": false
      }
    ],
    "created_at": "2026-06-20T08:30:00+08:00"
  }
}
```

---

## 8. 获取登录会话列表

```
GET /api/v1/session
```

### 请求头

```
Authorization: Bearer <access_token>
```

### 请求示例

```bash
curl "http://localhost:8090/api/v1/session" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."
```

### 响应示例

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "device_os": "macos",
      "device_type": "desktop",
      "browser": "chrome",
      "ip": "192.168.1.100",
      "login_at": "2026-06-24T09:00:00+08:00"
    },
    {
      "id": 2,
      "device_os": "ios",
      "device_type": "mobile",
      "browser": "safari",
      "ip": "192.168.1.101",
      "login_at": "2026-06-23T18:30:00+08:00"
    }
  ]
}
```
