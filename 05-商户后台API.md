# 商户后台 API

商户（Merchant）后台管理接口。所有请求需要携带 `Authorization: Bearer <token>` 头部及商户域上下文。

> **Logo 上传已迁移**：商户 Logo 上传功能已移至独立的图片上传服务（`service/image`），不再通过商户 API 处理。请调用 `POST /image/upload`（端口 8091）。详见 [08-图片上传服务API.md](./08-图片上传服务API.md)。

> **字段命名变更**：本版起所有 API 响应中的 JSON 字段名改为 PascalCase（如 `UserID`、`CreatedAt`）。

---

## 基础路径

```
/api/v1/merchant
```

---

## 商品管理 (Products)

Base: `/api/v1/merchant/product`

---

### 1. 商品列表

分页获取当前商户的商品列表，支持按状态和分类筛选。

- **URL**: `GET /api/v1/merchant/product`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数 (Query)**:

| 参数 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `Page` | integer | 否 | 1 | 页码 |
| `PageSize` | integer | 否 | 20 | 每页条数 |
| `Status` | string | 否 | — | 商品状态（如 `on` / `off`） |
| `CategoryID` | integer | 否 | — | 分类 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Pagination": {
    "Per": 20,
    "Count": 42,
    "Page": 3,
    "Current": 1,
    "Next": 2,
    "Previous": 0,
    "Order": "id:desc"
  },
  "Body": [
    {
      "ID": 101,
      "Name": "东北大米 50kg",
      "Price": 12000,
      "Stock": 200,
      "CategoryID": 5,
      "Brand": "五常",
      "SKU": "NCDM-50KG-001",
      "Description": "优质东北五常大米，50kg装",
      "Images": ["https://cdn.example.com/rice_01.jpg"],
      "Status": "on",
      "CreatedAt": "2025-01-01T08:00:00Z"
    }
  ]
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/product?page=1&page_size=20&status=on&category_id=5" \
  -H "Authorization: Bearer <token>"
```

---

### 2. 商品详情

获取指定商品的详细信息。

- **URL**: `GET /api/v1/merchant/product/:id`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 商品 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 101,
  "Name": "东北大米 50kg",
  "Price": 12000,
  "Stock": 200,
  "CategoryID": 5,
  "Brand": "五常",
  "SKU": "NCDM-50KG-001",
  "Description": "优质东北五常大米，50kg装",
  "Images": ["https://cdn.example.com/rice_01.jpg"],
  "Status": "on",
  "CreatedAt": "2025-01-01T08:00:00Z",
  "UpdatedAt": "2025-01-10T12:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/product/101" \
  -H "Authorization: Bearer <token>"
```

---

### 3. 创建商品

新增一个商品到商户。

- **URL**: `POST /api/v1/merchant/product`
- **Method**: `POST`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 是 | 商品名称 |
| `price` | integer | 是 | 价格（**分**为单位） |
| `stock` | integer | 是 | 库存数量 |
| `CategoryID` | integer | 是 | 分类 ID |
| `brand` | string | 否 | 品牌 |
| `sku` | string | 是 | SKU 编码 |
| `description` | string | 否 | 商品描述 |
| `images` | array of string | 否 | 图片 URL 数组（JSON 数组） |

**请求示例**:

```json
{
  "Name": "东北大米 50kg",
  "price": 12000,
  "stock": 200,
  "CategoryID": 5,
  "brand": "五常",
  "sku": "NCDM-50KG-001",
  "Description": "优质东北五常大米，50kg装",
  "images": ["https://cdn.example.com/rice_01.jpg"]
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 102,
  "Name": "东北大米 50kg",
  "Price": 12000,
  "Stock": 200,
  "CategoryID": 5,
  "Brand": "五常",
  "SKU": "NCDM-50KG-001",
  "Description": "优质东北五常大米，50kg装",
  "Images": ["https://cdn.example.com/rice_01.jpg"],
  "Status": "on",
  "CreatedAt": "2025-01-16T10:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X POST "https://api.example.com/api/v1/merchant/product" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "东北大米 50kg",
    "price": 12000,
    "stock": 200,
    "CategoryID": 5,
    "brand": "五常",
    "sku": "NCDM-50KG-001",
    "Description": "优质东北五常大米，50kg装",
    "images": ["https://cdn.example.com/rice_01.jpg"]
  }'
```

---

### 4. 更新商品

更新指定商品的信息。

- **URL**: `PUT /api/v1/merchant/product/:id`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 商品 ID |

**请求体 (JSON)**: 支持部分更新，与创建时的字段一致。

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 否 | 商品名称 |
| `price` | integer | 否 | 价格（分） |
| `stock` | integer | 否 | 库存数量 |
| `CategoryID` | integer | 否 | 分类 ID |
| `brand` | string | 否 | 品牌 |
| `sku` | string | 否 | SKU 编码 |
| `description` | string | 否 | 商品描述 |
| `images` | array of string | 否 | 图片 URL 数组 |

**请求示例**:

```json
{
  "price": 11000,
  "stock": 250
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 101,
  "Name": "东北大米 50kg",
  "price": 11000,
  "stock": 250,
  "CategoryID": 5,
  "brand": "五常",
  "sku": "NCDM-50KG-001",
  "Description": "优质东北五常大米，50kg装",
  "images": ["https://cdn.example.com/rice_01.jpg"],
  "Status": "on",
  "UpdatedAt": "2025-01-16T11:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/product/101" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "price": 11000,
    "stock": 250
  }'
```

---

### 5. 删除商品

删除指定商品（软删除或硬删除）。

- **URL**: `DELETE /api/v1/merchant/product/:id`
- **Method**: `DELETE`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 商品 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Product deleted successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X DELETE "https://api.example.com/api/v1/merchant/product/101" \
  -H "Authorization: Bearer <token>"
```

---

## 订单管理 (Orders)

Base: `/api/v1/merchant/order`

---

### 6. 订单列表

分页获取商户的订单列表，支持状态、关键词、日期范围筛选。

- **URL**: `GET /api/v1/merchant/order`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数 (Query)**:

| 参数 | 类型 | 必填 | 默认值 | 描述 |
|------|------|------|--------|------|
| `Page` | integer | 否 | 1 | 页码 |
| `PageSize` | integer | 否 | 20 | 每页条数 |
| `Status` | string | 否 | — | 订单状态 |
| `Keyword` | string | 否 | — | 搜索关键词（订单号/商品名等） |
| `StartDate` | string | 否 | — | 开始日期 (YYYY-MM-DD) |
| `EndDate` | string | 否 | — | 结束日期 (YYYY-MM-DD) |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Pagination": {
    "Per": 20,
    "Count": 150,
    "Page": 8,
    "Current": 1,
    "Next": 2,
    "Previous": 0,
    "Order": "id:desc"
  },
  "Body": [
    {
      "ID": 2001,
      "OrderNo": "ORD202501150001",
      "TotalAmount": 6000000,
      "Status": "paid",
      "UserName": "张三",
      "CreatedAt": "2025-01-15T12:00:00Z"
    }
  ]
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/order?page=1&page_size=20&status=paid&start_date=2025-01-01&end_date=2025-01-15" \
  -H "Authorization: Bearer <token>"
```

---

### 7. 订单详情

获取指定订单的详细信息。

- **URL**: `GET /api/v1/merchant/order/:id`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 订单 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 2001,
  "OrderNo": "ORD202501150001",
  "TotalAmount": 6000000,
  "Status": "paid",
  "UserName": "张三",
  "ShippingAddress": "北京市朝阳区建国路88号",
  "remark": "请尽快发货",
  "CreatedAt": "2025-01-15T12:00:00Z",
  "items": [
    {
      "ProductID": 101,
      "ProductName": "东北大米 50kg",
      "quantity": 500,
      "UnitPrice": 12000
    }
  ]
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/order/2001" \
  -H "Authorization: Bearer <token>"
```

---

### 8. 更新订单状态

更新指定订单的状态。

- **URL**: `PUT /api/v1/merchant/order/:id/status`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 订单 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `status` | string | 是 | 目标订单状态（如 `confirmed` / `processing` / `completed`） |

**请求示例**:

```json
{
  "Status": "processing"
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Order status updated successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/order/2001/status" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Status": "processing"
  }'
```

---

### 9. 发货

为指定订单进行发货操作，填写物流信息。

- **URL**: `PUT /api/v1/merchant/order/:id/ship`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 订单 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `TrackingNumber` | string | 是 | 快递单号 |
| `Carrier` | string | 是 | 快递公司（如 `顺丰` / `圆通` / `中通`） |

**请求示例**:

```json
{
  "TrackingNumber": "SF1234567890",
  "Carrier": "顺丰"
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 2001,
  "Status": "shipped",
  "TrackingNumber": "SF1234567890",
  "Carrier": "顺丰",
  "Message": "Order shipped successfully"
}
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/order/2001/ship" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "TrackingNumber": "SF1234567890",
    "Carrier": "顺丰"
  }'
```

---

### 10. 取消订单

取消指定订单。

- **URL**: `PUT /api/v1/merchant/order/:id/cancel`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 订单 ID |

**请求参数**: 无请求体

**响应示例**:

```json
{
  "ID": 2001,
  "Status": "cancelled",
  "Message": "Order cancelled successfully"
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/order/2001/cancel" \
  -H "Authorization: Bearer <token>"
```

---

### 11. 获取订单商品项

获取指定订单下的所有商品项。

- **URL**: `GET /api/v1/merchant/order/:id/item`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 订单 ID |

**响应示例**:

```json
[
  {
    "ID": 301,
    "OrderID": 2001,
    "ProductID": 101,
    "ProductName": "东北大米 50kg",
    "quantity": 500,
    "UnitPrice": 12000,
    "subtotal": 6000000
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/order/2001/item" \
  -H "Authorization: Bearer <token>"
```

---

### 12. 订单统计

获取商户的订单统计数据。

- **URL**: `GET /api/v1/merchant/order/statistic`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**: 可选日期范围

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `StartDate` | string | 否 | 开始日期 (YYYY-MM-DD) |
| `EndDate` | string | 否 | 结束日期 (YYYY-MM-DD) |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TotalOrders": 150,
  "TotalAmount": 900000000,
  "PendingCount": 10,
  "PaidCount": 30,
  "ShippedCount": 50,
  "CompletedCount": 55,
  "CancelledCount": 5
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/order/statistic?start_date=2025-01-01&end_date=2025-01-31" \
  -H "Authorization: Bearer <token>"
```

---

## 库存管理 (Inventory)

Base: `/api/v1/merchant/inventory`

---

### 13. 库存列表

获取商户的所有库存记录列表。

- **URL**: `GET /api/v1/merchant/inventory`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求参数**: 无

**响应示例**:

```json
[
  {
    "ProductID": 101,
    "ProductName": "东北大米 50kg",
    "stock": 200,
    "LowStockThreshold": 50,
    "UpdatedAt": "2025-01-16T10:00:00Z"
  },
  {
    "ProductID": 102,
    "ProductName": "有机鸡蛋 30枚装",
    "stock": 500,
    "LowStockThreshold": 100,
    "UpdatedAt": "2025-01-16T10:00:00Z"
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/inventory" \
  -H "Authorization: Bearer <token>"
```

---

### 14. 按商品查看库存

获取指定商品的库存信息。

- **URL**: `GET /api/v1/merchant/inventory/:productId`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `productId` | integer | 商品 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ProductID": 101,
  "ProductName": "东北大米 50kg",
  "stock": 200,
  "LowStockThreshold": 50,
  "UpdatedAt": "2025-01-16T10:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/inventory/101" \
  -H "Authorization: Bearer <token>"
```

---

### 15. 更新库存

更新指定商品的库存数量。

- **URL**: `PUT /api/v1/merchant/inventory/:productId`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `productId` | integer | 商品 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `stock` | integer | 是 | 最新库存数量 |

**请求示例**:

```json
{
  "stock": 180
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Inventory updated successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/inventory/101" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "stock": 180
  }'
```

---

### 16. 低库存预警

获取所有低库存商品的预警列表。

- **URL**: `GET /api/v1/merchant/inventory/low-stock`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求参数**: 无

**响应示例**:

```json
[
  {
    "ProductID": 103,
    "ProductName": "精选面粉 25kg",
    "stock": 10,
    "LowStockThreshold": 30,
    "UpdatedAt": "2025-01-16T08:00:00Z"
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/inventory/low-stock" \
  -H "Authorization: Bearer <token>"
```

---

### 17. 库存警报

获取库存相关的所有警报信息。

- **URL**: `GET /api/v1/merchant/inventory/alert`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求参数**: 无

**响应示例**:

```json
[
  {
    "ProductID": 103,
    "ProductName": "精选面粉 25kg",
    "AlertType": "low_stock",
    "Message": "库存仅剩 10，低于预警线 30",
    "CreatedAt": "2025-01-16T08:00:00Z"
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/inventory/alert" \
  -H "Authorization: Bearer <token>"
```

---

## 分类管理 (Categories)

Base: `/api/v1/merchant/category`

---

### 18. 分类列表

获取商户的所有商品分类列表。

- **URL**: `GET /api/v1/merchant/category`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**响应示例**:

```json
[
  {
    "ID": 1,
    "Name": "粮油调味",
    "ParentID": null,
    "SortOrder": 1,
    "CreatedAt": "2025-01-01T00:00:00Z"
  },
  {
    "ID": 5,
    "Name": "大米",
    "ParentID": 1,
    "SortOrder": 1,
    "CreatedAt": "2025-01-01T00:00:00Z"
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/category" \
  -H "Authorization: Bearer <token>"
```

---

### 19. 分类详情

获取指定分类的详细信息。

- **URL**: `GET /api/v1/merchant/category/:id`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 分类 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 5,
  "Name": "大米",
  "ParentID": 1,
  "SortOrder": 1,
  "CreatedAt": "2025-01-01T00:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/category/5" \
  -H "Authorization: Bearer <token>"
```

---

### 20. 创建分类

新增一个商品分类。

- **URL**: `POST /api/v1/merchant/category`
- **Method**: `POST`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 是 | 分类名称 |
| `ParentID` | integer | 否 | 父分类 ID（null 为顶级分类） |
| `SortOrder` | integer | 否 | 排序序号 |

**请求示例**:

```json
{
  "Name": "有机食品",
  "ParentID": null,
  "SortOrder": 10
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 10,
  "Name": "有机食品",
  "ParentID": null,
  "SortOrder": 10,
  "CreatedAt": "2025-01-16T14:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X POST "https://api.example.com/api/v1/merchant/category" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "有机食品",
    "ParentID": null,
    "SortOrder": 10
  }'
```

---

### 21. 更新分类

更新指定分类的信息。

- **URL**: `PUT /api/v1/merchant/category/:id`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 分类 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 否 | 分类名称 |
| `ParentID` | integer | 否 | 父分类 ID |
| `SortOrder` | integer | 否 | 排序序号 |

**请求示例**:

```json
{
  "Name": "有机食品专区",
  "SortOrder": 5
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 10,
  "Name": "有机食品专区",
  "ParentID": null,
  "SortOrder": 5,
  "UpdatedAt": "2025-01-16T14:30:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/category/10" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "有机食品专区",
    "SortOrder": 5
  }'
```

---

### 22. 删除分类

删除指定分类。

- **URL**: `DELETE /api/v1/merchant/category/:id`
- **Method**: `DELETE`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 分类 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Category deleted successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X DELETE "https://api.example.com/api/v1/merchant/category/10" \
  -H "Authorization: Bearer <token>"
```

---

### 23. 分类树

获取商户分类的树形结构。

- **URL**: `GET /api/v1/merchant/category/tree`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**响应示例**:

```json
[
  {
    "ID": 1,
    "Name": "粮油调味",
    "SortOrder": 1,
    "children": [
      {
        "ID": 5,
        "Name": "大米",
        "SortOrder": 1,
        "children": []
      },
      {
        "ID": 6,
        "Name": "食用油",
        "SortOrder": 2,
        "children": []
      }
    ]
  },
  {
    "ID": 2,
    "Name": "生鲜食材",
    "SortOrder": 2,
    "children": [
      {
        "ID": 7,
        "Name": "蔬菜",
        "SortOrder": 1,
        "children": []
      }
    ]
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/category/tree" \
  -H "Authorization: Bearer <token>"
```

---

## 商户设置 (Merchant Settings)

Base: `/api/v1/merchant/merchant`

---

### 24. 获取商户信息

获取当前商户的基本信息。

- **URL**: `GET /api/v1/merchant/merchant`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 3,
  "Name": "好味道食材有限公司",
  "Phone": "13800138000",
  "Address": "北京市海淀区中关村大街1号",
  "Logo": "https://cdn.example.com/merchant_logo.png",
  "Status": "active",
  "CreatedAt": "2024-06-01T00:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/merchant" \
  -H "Authorization: Bearer <token>"
```

---

### 25. 更新商户信息

更新当前商户的基本信息。

- **URL**: `PUT /api/v1/merchant/merchant`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 否 | 商户名称 |
| `ContactName` | string | 否 | 联系人姓名 |
| `ContactPhone` | string | 否 | 联系电话 |
| `address` | string | 否 | 地址 |

**请求示例**:

```json
{
  "ContactName": "李经理",
  "ContactPhone": "13900139000"
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 3,
  "Name": "好味道食材有限公司",
  "Phone": "13900139000",
  "Address": "北京市海淀区中关村大街1号",
  "Logo": "https://cdn.example.com/merchant_logo.png",
  "Status": "active",
  "UpdatedAt": "2025-01-16T15:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/merchant" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "ContactName": "李经理",
    "ContactPhone": "13900139000"
  }'
```

---

### 26. 创建商户

创建一个新的商户账号。

- **URL**: `POST /api/v1/merchant/merchant`
- **Method**: `POST`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 是 | 商户名称 |
| `ContactName` | string | 是 | 联系人姓名 |
| `ContactPhone` | string | 是 | 联系电话 |
| `address` | string | 否 | 地址 |

**请求示例**:

```json
{
  "Name": "新鲜农场直供",
  "ContactName": "赵总",
  "ContactPhone": "13700137000",
  "address": "上海市浦东新区张江路100号"
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 10,
  "Name": "新鲜农场直供",
  "ContactName": "赵总",
  "ContactPhone": "13700137000",
  "address": "上海市浦东新区张江路100号",
  "Status": "active",
  "CreatedAt": "2025-01-16T16:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X POST "https://api.example.com/api/v1/merchant/merchant" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "新鲜农场直供",
    "ContactName": "赵总",
    "ContactPhone": "13700137000",
    "address": "上海市浦东新区张江路100号"
  }'
```

---

## 数据分析 (Analytics)

Base: `/api/v1/merchant/analytic`

---

### 28. 销售分析

获取销售相关的分析数据。

- **URL**: `GET /api/v1/merchant/analytic/sales`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**:

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `StartDate` | string | 否 | 开始日期 (YYYY-MM-DD) |
| `EndDate` | string | 否 | 结束日期 (YYYY-MM-DD) |
| `Period` | string | 否 | 统计周期（daily / weekly / monthly） |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TotalSales": 900000000,
  "OrderCount": 150,
  "AvgOrderValue": 6000000,
  "DailyData": [
    {"date": "2025-01-15", "sales": 30000000, "orders": 5},
    {"date": "2025-01-16", "sales": 45000000, "orders": 8}
  ]
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/analytic/sales?start_date=2025-01-01&end_date=2025-01-31&period=daily" \
  -H "Authorization: Bearer <token>"
```

---

### 29. 收入分析

获取收入相关的分析数据。

- **URL**: `GET /api/v1/merchant/analytic/revenue`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**:

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `StartDate` | string | 否 | 开始日期 |
| `EndDate` | string | 否 | 结束日期 |
| `Period` | string | 否 | 统计周期 |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TotalRevenue": 850000000,
  "Commission": 50000000,
  "NetIncome": 800000000,
  "DailyData": [
    {"date": "2025-01-15", "revenue": 28000000, "Commission": 1500000}
  ]
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/analytic/revenue" \
  -H "Authorization: Bearer <token>"
```

---

### 30. 客户分析

获取客户相关的分析数据。

- **URL**: `GET /api/v1/merchant/analytic/customer`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**:

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `StartDate` | string | 否 | 开始日期 |
| `EndDate` | string | 否 | 结束日期 |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TotalCustomers": 85,
  "NewCustomers": 12,
  "RepeatCustomers": 25,
  "AvgPurchaseFrequency": 2.3
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/analytic/customer" \
  -H "Authorization: Bearer <token>"
```

---

### 31. 商品分析

获取商品相关的分析数据。

- **URL**: `GET /api/v1/merchant/analytic/product`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**:

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `StartDate` | string | 否 | 开始日期 |
| `EndDate` | string | 否 | 结束日期 |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TopProducts": [
    {"ProductID": 101, "Name": "东北大米 50kg", "sales": 500, "revenue": 6000000},
    {"ProductID": 102, "Name": "有机鸡蛋 30枚装", "sales": 300, "revenue": 1350000}
  ],
  "TotalProducts": 42,
  "ActiveProducts": 38
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/analytic/product" \
  -H "Authorization: Bearer <token>"
```

---

### 32. 总览分析

获取商户经营总览分析数据。

- **URL**: `GET /api/v1/merchant/analytic/overview`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**查询参数**: 无

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "TodayOrders": 5,
  "TodayRevenue": 30000000,
  "MonthOrders": 150,
  "MonthRevenue": 900000000,
  "PendingOrders": 10,
  "LowStockCount": 3,
  "TotalProducts": 42
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/analytic/overview" \
  -H "Authorization: Bearer <token>"
```

---

## 角色管理 (Roles)

Base: `/api/v1/merchant/role`

---

### 33. 角色列表

获取商户的所有角色列表。

- **URL**: `GET /api/v1/merchant/role`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**响应示例**:

```json
[
  {
    "ID": 1,
    "Name": "管理员",
    "code": "admin",
    "Description": "拥有所有权限",
    "permissions": ["product.*", "order.*", "inventory.*", "analytic.*"],
    "Status": "active",
    "CreatedAt": "2024-01-01T00:00:00Z"
  },
  {
    "ID": 2,
    "Name": "运营",
    "code": "operator",
    "Description": "商品和订单管理",
    "permissions": ["product.read", "product.write", "order.read", "order.write"],
    "Status": "active",
    "CreatedAt": "2024-01-01T00:00:00Z"
  }
]
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/role" \
  -H "Authorization: Bearer <token>"
```

---

### 34. 角色详情

获取指定角色的详细信息。

- **URL**: `GET /api/v1/merchant/role/:id`
- **Method**: `GET`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 角色 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 1,
  "Name": "管理员",
  "code": "admin",
  "Description": "拥有所有权限",
  "permissions": ["product.*", "order.*", "inventory.*", "analytic.*"],
  "Status": "active",
  "CreatedAt": "2024-01-01T00:00:00Z",
  "UpdatedAt": "2024-06-01T00:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X GET "https://api.example.com/api/v1/merchant/role/1" \
  -H "Authorization: Bearer <token>"
```

---

### 35. 创建角色

新增一个角色。

- **URL**: `POST /api/v1/merchant/role`
- **Method**: `POST`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 是 | 角色名称 |
| `code` | string | 是 | 角色编码 |
| `description` | string | 否 | 角色描述 |
| `permissions` | array of string | 否 | 权限列表 |
| `status` | string | 否 | 状态（active / inactive） |

**请求示例**:

```json
{
  "Name": "仓库管理员",
  "code": "warehouse",
  "Description": "管理库存和发货",
  "permissions": ["inventory.read", "inventory.write", "order.read", "order.ship"]
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 3,
  "Name": "仓库管理员",
  "code": "warehouse",
  "Description": "管理库存和发货",
  "permissions": ["inventory.read", "inventory.write", "order.read", "order.ship"],
  "Status": "active",
  "CreatedAt": "2025-01-16T17:00:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X POST "https://api.example.com/api/v1/merchant/role" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "仓库管理员",
    "code": "warehouse",
    "Description": "管理库存和发货",
    "permissions": ["inventory.read", "inventory.write", "order.read", "order.ship"]
  }'
```

---

### 36. 更新角色

更新指定角色的信息。

- **URL**: `PUT /api/v1/merchant/role/:id`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 角色 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `name` | string | 否 | 角色名称 |
| `description` | string | 否 | 角色描述 |
| `permissions` | array of string | 否 | 权限列表 |
| `status` | string | 否 | 状态 |

**请求示例**:

```json
{
  "Name": "高级仓库管理员",
  "permissions": ["inventory.*", "order.read", "order.ship"]
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": {
  "ID": 3,
  "Name": "高级仓库管理员",
  "code": "warehouse",
  "Description": "管理库存和发货",
  "permissions": ["inventory.*", "order.read", "order.ship"],
  "Status": "active",
  "UpdatedAt": "2025-01-16T17:30:00Z"
}
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/role/3" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "Name": "高级仓库管理员",
    "permissions": ["inventory.*", "order.read", "order.ship"]
  }'
```

---

### 37. 删除角色

删除指定角色。

- **URL**: `DELETE /api/v1/merchant/role/:id`
- **Method**: `DELETE`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 角色 ID |

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Role deleted successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X DELETE "https://api.example.com/api/v1/merchant/role/3" \
  -H "Authorization: Bearer <token>"
```

---

### 38. 角色权限分配（额外）

为角色分配/更新权限列表（部分实现中可能合并到更新角色中）。

- **URL**: `PUT /api/v1/merchant/role/:id/permission`
- **Method**: `PUT`
- **Auth**: `Authorization: Bearer <token>` + 商户域上下文

**路径参数**:

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | integer | 角色 ID |

**请求体 (JSON)**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| `permissions` | array of string | 是 | 权限标识列表 |

**请求示例**:

```json
{
  "permissions": ["product.read", "product.write", "order.read"]
}
```

**响应示例**:

```json
{
  "Head": {
    "Code": "1000",
    "msg": "Permissions updated successfully"
  }
}
```

**Curl 示例**:

```bash
curl -X PUT "https://api.example.com/api/v1/merchant/role/2/permission" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "permissions": ["product.read", "product.write", "order.read"]
  }'
```