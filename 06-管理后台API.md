# 管理后台 API

> 所有本组接口需要管理员身份认证且拥有 **平台管理员 (platform admin)** 角色。

**基础路径:** `/api/v1/admin`

---

## 目录

- [Dashboard（仪表盘）](#dashboard仪表盘)
- [Merchants（商户管理）](#merchants商户管理)
- [Users（用户管理）](#users用户管理)
- [Roles（角色管理）](#roles角色管理)
- [Qualifications（资质审核）](#qualifications资质审核)
- [Intents（意向管理）](#intents意向管理)
- [Products（产品管理）](#products产品管理)
- [Categories（分类管理）](#categories分类管理)

---

## Dashboard（仪表盘）

Base: `/api/v1/admin`

### 1. GET /dashboard — 仪表盘概览

返回平台核心指标概览数据。

**完整 URL:** `GET /api/v1/admin/dashboard`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**请求参数:** 无

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "total_users": 1280,
    "total_merchants": 45,
    "total_products": 3200,
    "total_orders": 15890,
    "today_revenue": 256800.00,
    "today_new_users": 32,
    "today_new_orders": 178,
    "pending_qualifications": 5
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/dashboard' \
  -H 'Authorization: Bearer <token>'
```

---

### 2. GET /stats — 系统统计

返回平台详细统计数据（支持时间范围过滤）。

**完整 URL:** `GET /api/v1/admin/stats`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| start_date | string | 否 | 起始日期，格式 `YYYY-MM-DD` |
| end_date | string | 否 | 结束日期，格式 `YYYY-MM-DD` |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "user_growth": { "2025-01-01": 10, "2025-01-02": 15 },
    "revenue_daily": { "2025-01-01": 120000, "2025-01-02": 135000 },
    "order_volume": { "2025-01-01": 95, "2025-01-02": 110 },
    "active_merchants": 42
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/stats?start_date=2025-01-01&end_date=2025-01-07' \
  -H 'Authorization: Bearer <token>'
```

---

## Merchants（商户管理）

Base: `/api/v1/admin/merchant`

### 3. GET / — 商户列表

分页获取所有商户。

**完整 URL:** `GET /api/v1/admin/merchant`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |
| status | string | 否 | 商户状态过滤（active / inactive / pending） |
| keyword | string | 否 | 搜索关键词（名称或编码） |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "name": "示例商户",
      "code": "MERCHANT001",
      "domain": "shop.example.com",
      "owner_id": 10,
      "phone": "13800000001",
      "status": "active",
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  ],
  "pagination": {
    "per": 20,
    "count": 45,
    "page": 3,
    "current": 1,
    "next": 2,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/merchant?page=1&page_size=20&status=active' \
  -H 'Authorization: Bearer <token>'
```

---

### 4. GET /:id — 获取商户详情

**完整 URL:** `GET /api/v1/admin/merchant/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 商户 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "示例商户",
    "code": "MERCHANT001",
    "domain": "shop.example.com",
    "owner_id": 10,
    "phone": "13800000001",
    "description": "主营数码产品",
    "address": "北京市朝阳区xx路100号",
    "province": "北京市",
    "city": "北京市",
    "district": "朝阳区",
    "status": "active",
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-15T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/merchant/1' \
  -H 'Authorization: Bearer <token>'
```

---

### 5. POST / — 创建商户

**完整 URL:** `POST /api/v1/admin/merchant`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | string | **是** | 商户名称 |
| code | string | **是** | 商户编码，唯一 |
| domain | string | **是** | 商户域名 |
| owner_id | integer | **是** | 负责人用户 ID |
| phone | string | 否 | 联系电话 |
| description | string | 否 | 商户简介 |
| address | string | 否 | 详细地址 |
| province | string | 否 | 省份 |
| city | string | 否 | 城市 |
| district | string | 否 | 区/县 |

**Request 示例:**

```json
{
  "name": "新商户",
  "code": "MERCHANT002",
  "domain": "newshop.example.com",
  "owner_id": 10,
  "phone": "13800000002",
  "description": "新入驻商户",
  "address": "上海市浦东新区xx路200号",
  "province": "上海市",
  "city": "上海市",
  "district": "浦东新区"
}
```

**Response `201 Created`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 2,
    "name": "新商户",
    "code": "MERCHANT002",
    "domain": "newshop.example.com",
    "owner_id": 10,
    "phone": "13800000002",
    "status": "active",
    "created_at": "2025-01-20T10:00:00Z",
    "updated_at": "2025-01-20T10:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/admin/merchant' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "新商户",
    "code": "MERCHANT002",
    "domain": "newshop.example.com",
    "owner_id": 10
  }'
```

---

### 6. PUT /:id — 更新商户

**完整 URL:** `PUT /api/v1/admin/merchant/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 商户 ID |

**请求体 (JSON):** 与创建商户相同，所有字段均为可选（按需更新）。

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "更新后的商户名称",
    "code": "MERCHANT001",
    "domain": "updated.example.com",
    "owner_id": 10,
    "status": "active",
    "updated_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/merchant/1' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "更新后的商户名称",
    "domain": "updated.example.com"
  }'
```

---

### 7. DELETE /:id — 删除商户

**完整 URL:** `DELETE /api/v1/admin/merchant/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 商户 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  }
}
```

**Curl 示例:**

```bash
curl -X DELETE 'https://api.example.com/api/v1/admin/merchant/1' \
  -H 'Authorization: Bearer <token>'
```

---

## Users（用户管理）

Base: `/api/v1/admin/user`

### 8. GET / — 用户列表

**完整 URL:** `GET /api/v1/admin/user`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |
| status | string | 否 | 用户状态过滤（active / inactive / banned） |
| keyword | string | 否 | 搜索关键词（用户名/手机号） |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "username": "user01",
      "phone": "13800000001",
      "status": "active",
      "avatar": "https://cdn.example.com/avatars/1.jpg",
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  ],
  "pagination": {
    "per": 20,
    "count": 1280,
    "page": 64,
    "current": 1,
    "next": 2,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/user?page=1&page_size=20&status=active&keyword=user01' \
  -H 'Authorization: Bearer <token>'
```

---

### 9. GET /:id — 获取用户详情

**完整 URL:** `GET /api/v1/admin/user/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 用户 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "username": "user01",
    "phone": "13800000001",
    "status": "active",
    "avatar": "https://cdn.example.com/avatars/1.jpg",
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-15T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/user/1' \
  -H 'Authorization: Bearer <token>'
```

---

### 10. PUT /:id — 更新用户

**完整 URL:** `PUT /api/v1/admin/user/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 用户 ID |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| username | string | 否 | 用户名 |
| phone | string | 否 | 手机号 |
| status | string | 否 | 状态（active / inactive / banned） |
| avatar | string | 否 | 头像 URL |

**Request 示例:**

```json
{
  "username": "newname",
  "status": "active"
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
    "id": 1,
    "username": "newname",
    "phone": "13800000001",
    "status": "active",
    "avatar": "https://cdn.example.com/avatars/1.jpg",
    "updated_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/user/1' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "username": "newname",
    "status": "active"
  }'
```

---

## Roles（角色管理）

Base: `/api/v1/admin/role`

### 11. GET / — 角色列表

**完整 URL:** `GET /api/v1/admin/role`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "name": "超级管理员",
      "description": "平台超级管理员",
      "merchant_id": null,
      "admin": true,
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    },
    {
      "id": 2,
      "name": "商户管理员",
      "description": "各商户的管理员角色",
      "merchant_id": 1,
      "admin": false,
      "created_at": "2025-01-05T00:00:00Z",
      "updated_at": "2025-01-05T00:00:00Z"
    }
  ],
  "pagination": {
    "per": 20,
    "count": 10,
    "page": 1,
    "current": 1,
    "next": 0,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/role?page=1&page_size=20' \
  -H 'Authorization: Bearer <token>'
```

---

### 12. GET /:id — 获取角色详情

**完整 URL:** `GET /api/v1/admin/role/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 角色 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "超级管理员",
    "description": "平台超级管理员",
    "merchant_id": null,
    "admin": true,
    "permissions": ["user:read", "user:write", "merchant:read", "merchant:write"],
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/role/1' \
  -H 'Authorization: Bearer <token>'
```

---

### 13. POST / — 创建角色

**完整 URL:** `POST /api/v1/admin/role`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | string | **是** | 角色名称 |
| merchant_id | integer | 否 | 关联商户 ID（为空则为平台级角色） |
| description | string | 否 | 角色描述 |
| admin | boolean | 否 | 是否管理员角色，默认 false |

**Request 示例:**

```json
{
  "name": "运营人员",
  "merchant_id": 1,
  "description": "商户运营角色",
  "admin": false
}
```

**Response `201 Created`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 3,
    "name": "运营人员",
    "merchant_id": 1,
    "description": "商户运营角色",
    "admin": false,
    "created_at": "2025-01-20T10:00:00Z",
    "updated_at": "2025-01-20T10:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/admin/role' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "运营人员",
    "merchant_id": 1,
    "description": "商户运营角色",
    "admin": false
  }'
```

---

### 14. PUT /:id — 更新角色

**完整 URL:** `PUT /api/v1/admin/role/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 角色 ID |

**请求体 (JSON):** 与创建角色相同，所有字段均为可选。

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 3,
    "name": "高级运营",
    "merchant_id": 1,
    "description": "升级后的角色",
    "admin": false,
    "updated_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/role/3' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "高级运营",
    "description": "升级后的角色"
  }'
```

---

### 15. DELETE /:id — 删除角色

**完整 URL:** `DELETE /api/v1/admin/role/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 角色 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  }
}
```

**Curl 示例:**

```bash
curl -X DELETE 'https://api.example.com/api/v1/admin/role/3' \
  -H 'Authorization: Bearer <token>'
```

---

## Qualifications（资质审核）

Base: `/api/v1/admin/qualification`

### 16. GET / — 资质列表

**完整 URL:** `GET /api/v1/admin/qualification`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| status | string | 否 | 审核状态（pending / approved / rejected） |
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |

**Response `200 OK`:**

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
      "merchant_name": "示例商户",
      "type": "business_license",
      "file_url": "https://cdn.example.com/qualifications/1.pdf",
      "status": "pending",
      "remark": null,
      "submitted_at": "2025-01-18T10:00:00Z",
      "reviewed_at": null
    }
  ],
  "pagination": {
    "per": 20,
    "count": 5,
    "page": 1,
    "current": 1,
    "next": 0,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/qualification?status=pending&page=1&page_size=20' \
  -H 'Authorization: Bearer <token>'
```

---

### 17. POST /:id/approve — 审核资质

**完整 URL:** `POST /api/v1/admin/qualification/:id/approve`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 资质 ID |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| action | string | **是** | 审核操作：`approve` 或 `reject` |
| remark | string | 否 | 审核备注/拒绝原因 |

**Request 示例:**

```json
{
  "action": "approve",
  "remark": "资质齐全，审核通过"
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
    "id": 1,
    "status": "approved",
    "remark": "资质齐全，审核通过",
    "reviewed_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/admin/qualification/1/approve' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "action": "approve",
    "remark": "资质齐全，审核通过"
  }'
```

---

## Intents（意向管理）

Base: `/api/v1/admin/intent`

### 18. GET / — 意向列表

**完整 URL:** `GET /api/v1/admin/intent`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| status | string | 否 | 意向状态（new / contacted / converted / closed） |
| keyword | string | 否 | 搜索关键词 |
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "name": "张三",
      "phone": "13800000001",
      "email": "zhangsan@example.com",
      "status": "new",
      "source": "website",
      "remark": "有意入驻平台",
      "created_at": "2025-01-19T08:00:00Z",
      "updated_at": "2025-01-19T08:00:00Z"
    }
  ],
  "pagination": {
    "per": 20,
    "count": 15,
    "page": 1,
    "current": 1,
    "next": 0,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/intent?status=new&page=1&page_size=20' \
  -H 'Authorization: Bearer <token>'
```

---

### 19. GET /:id — 意向详情

**完整 URL:** `GET /api/v1/admin/intent/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 意向 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "张三",
    "phone": "13800000001",
    "email": "zhangsan@example.com",
    "company": "某某科技有限公司",
    "status": "new",
    "source": "website",
    "remark": "有意入驻平台",
    "contact_history": [
      {
        "contacted_at": "2025-01-19T10:00:00Z",
        "note": "已发送入驻指南邮件"
      }
    ],
    "created_at": "2025-01-19T08:00:00Z",
    "updated_at": "2025-01-19T10:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/intent/1' \
  -H 'Authorization: Bearer <token>'
```

---

## Products（产品管理）

Base: `/api/v1/admin/product`

### 20. GET / — 产品列表

**完整 URL:** `GET /api/v1/admin/product`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| page | integer | 否 | 页码，默认 1 |
| page_size | integer | 否 | 每页条数，默认 20 |
| status | string | 否 | 产品状态（active / inactive） |
| category_id | integer | 否 | 分类 ID 过滤 |
| merchant_id | integer | 否 | 商户 ID 过滤 |
| keyword | string | 否 | 搜索关键词 |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": [
    {
      "id": 1,
      "name": "示例商品",
      "merchant_id": 1,
      "merchant_name": "示例商户",
      "category_id": 2,
      "category_name": "电子产品",
      "price": 9999,
      "status": "active",
      "stock": 100,
      "created_at": "2025-01-10T00:00:00Z",
      "updated_at": "2025-01-15T12:00:00Z"
    }
  ],
  "pagination": {
    "per": 20,
    "count": 3200,
    "page": 160,
    "current": 1,
    "next": 2,
    "previous": 0,
    "order": "id:desc"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/product?page=1&page_size=20&status=active' \
  -H 'Authorization: Bearer <token>'
```

---

### 21. GET /:id — 产品详情

**完整 URL:** `GET /api/v1/admin/product/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 产品 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "示例商品",
    "merchant_id": 1,
    "merchant_name": "示例商户",
    "category_id": 2,
    "category_name": "电子产品",
    "description": "这是一款示例商品",
    "images": ["https://cdn.example.com/products/1/1.jpg"],
    "price": 9999,
    "status": "active",
    "stock": 100,
    "sales": 50,
    "created_at": "2025-01-10T00:00:00Z",
    "updated_at": "2025-01-15T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/product/1' \
  -H 'Authorization: Bearer <token>'
```

---

### 22. POST / — 创建产品

**完整 URL:** `POST /api/v1/admin/product`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | string | **是** | 产品名称 |
| merchant_id | integer | **是** | 所属商户 ID |
| category_id | integer | **是** | 分类 ID |
| price | integer | **是** | 价格（单位：分） |
| description | string | 否 | 产品描述 |
| images | string[] | 否 | 产品图片 URL 列表 |
| stock | integer | 否 | 库存数量，默认 0 |
| status | string | 否 | 状态，默认 active |

**Request 示例:**

```json
{
  "name": "新产品",
  "merchant_id": 1,
  "category_id": 2,
  "price": 19900,
  "description": "这是新产品描述",
  "images": ["https://cdn.example.com/products/new/1.jpg"],
  "stock": 200,
  "status": "active"
}
```

**Response `201 Created`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 100,
    "name": "新产品",
    "merchant_id": 1,
    "category_id": 2,
    "price": 19900,
    "status": "active",
    "stock": 200,
    "created_at": "2025-01-20T10:00:00Z",
    "updated_at": "2025-01-20T10:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/admin/product' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "新产品",
    "merchant_id": 1,
    "category_id": 2,
    "price": 19900,
    "description": "这是新产品描述",
    "images": ["https://cdn.example.com/products/new/1.jpg"],
    "stock": 200,
    "status": "active"
  }'
```

---

### 23. PUT /:id — 更新产品

**完整 URL:** `PUT /api/v1/admin/product/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 产品 ID |

**请求体 (JSON):** 与创建产品相同，所有字段均为可选。

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 100,
    "name": "更新后的产品",
    "price": 23900,
    "updated_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/product/100' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "更新后的产品",
    "price": 23900
  }'
```

---

### 24. DELETE /:id — 删除产品

**完整 URL:** `DELETE /api/v1/admin/product/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 产品 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  }
}
```

**Curl 示例:**

```bash
curl -X DELETE 'https://api.example.com/api/v1/admin/product/100' \
  -H 'Authorization: Bearer <token>'
```

---

## Categories（分类管理）

Base: `/api/v1/admin/category`

### 25. GET / — 分类列表

**完整 URL:** `GET /api/v1/admin/category`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**Query 参数:**

| 参数 | 类型 | 必填 | 说明 |
|---|---|---|---|
| parent_id | integer | 否 | 父分类 ID（不传则返回顶级分类） |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "pagination": {
    "per": 20,
    "count": 20,
    "page": 1,
    "current": 1,
    "next": 0,
    "previous": 0,
    "order": "id:desc"
  },
  "body": [
    {
      "id": 1,
      "name": "电子产品",
      "parent_id": null,
      "sort_order": 1,
      "status": "active",
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    },
    {
      "id": 2,
      "name": "手机通讯",
      "parent_id": 1,
      "sort_order": 1,
      "status": "active",
      "created_at": "2025-01-02T00:00:00Z",
      "updated_at": "2025-01-02T00:00:00Z"
    }
  ]
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/category' \
  -H 'Authorization: Bearer <token>'
```

---

### 26. GET /:id — 分类详情

**完整 URL:** `GET /api/v1/admin/category/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 分类 ID |

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 1,
    "name": "电子产品",
    "parent_id": null,
    "sort_order": 1,
    "status": "active",
    "children": [
      {
        "id": 2,
        "name": "手机通讯",
        "parent_id": 1,
        "sort_order": 1,
        "status": "active"
      }
    ],
    "created_at": "2025-01-01T00:00:00Z",
    "updated_at": "2025-01-01T00:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X GET 'https://api.example.com/api/v1/admin/category/1' \
  -H 'Authorization: Bearer <token>'
```

---

### 27. POST / — 创建分类

**完整 URL:** `POST /api/v1/admin/category`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| name | string | **是** | 分类名称 |
| parent_id | integer | 否 | 父分类 ID（不传则为顶级分类） |
| sort_order | integer | 否 | 排序序号，默认 0 |
| status | string | 否 | 状态，默认 active |

**Request 示例:**

```json
{
  "name": "智能家居",
  "parent_id": 1,
  "sort_order": 3,
  "status": "active"
}
```

**Response `201 Created`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 21,
    "name": "智能家居",
    "parent_id": 1,
    "sort_order": 3,
    "status": "active",
    "created_at": "2025-01-20T10:00:00Z",
    "updated_at": "2025-01-20T10:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X POST 'https://api.example.com/api/v1/admin/category' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "智能家居",
    "parent_id": 1,
    "sort_order": 3
  }'
```

---

### 28. PUT /:id — 更新分类

**完整 URL:** `PUT /api/v1/admin/category/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 分类 ID |

**请求体 (JSON):** 与创建分类相同，所有字段均为可选。

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  },
  "body": {
    "id": 21,
    "name": "智能家居设备",
    "sort_order": 2,
    "updated_at": "2025-01-20T12:00:00Z"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/category/21' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "智能家居设备",
    "sort_order": 2
  }'
```

---

### 29. DELETE /:id — 删除分类

**完整 URL:** `DELETE /api/v1/admin/category/:id`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |

**路径参数:**

| 参数 | 类型 | 说明 |
|---|---|---|
| id | integer | 分类 ID |

> ⚠️ 删除分类前需确保该分类下没有子分类和关联产品，否则可能返回错误。

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  }
}
```

**Curl 示例:**

```bash
curl -X DELETE 'https://api.example.com/api/v1/admin/category/21' \
  -H 'Authorization: Bearer <token>'
```

---

### 30. (可选) 批量操作 — 批量更新分类排序

**完整 URL:** `PUT /api/v1/admin/category/batch-order`

**请求头:**

| Header | 值 |
|---|---|
| Authorization | Bearer \<token\> |
| Content-Type | application/json |

**请求体 (JSON):**

```json
{
  "items": [
    { "id": 1, "sort_order": 1 },
    { "id": 2, "sort_order": 2 },
    { "id": 21, "sort_order": 3 }
  ]
}
```

**Response `200 OK`:**

```json
{
  "head": {
    "code": "1000",
    "msg": "Success"
  }
}
```

**Curl 示例:**

```bash
curl -X PUT 'https://api.example.com/api/v1/admin/category/batch-order' \
  -H 'Authorization: Bearer <token>' \
  -H 'Content-Type: application/json' \
  -d '{
    "items": [
      { "id": 1, "sort_order": 1 },
      { "id": 2, "sort_order": 2 }
    ]
  }'
```

---

## 通用错误码

| HTTP 状态码 | code | message | 说明 |
|---|---|---|---|
| 401 | 40100 | Unauthorized | 未认证或 token 过期 |
| 403 | 40300 | Forbidden | 无权限（非管理员） |
| 404 | 40400 | Not Found | 资源不存在 |
| 422 | 42200 | Validation Error | 请求参数校验失败 |
| 500 | 50000 | Internal Server Error | 服务器内部错误 |

**错误响应示例:**

```json
{
  "head": {
    "code": "1001",
    "msg": "名称不能为空"
  }
}
```
