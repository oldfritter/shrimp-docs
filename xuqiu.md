# Shrimp 采购平台 — 项目需求文档

> 文档版本：v1.0  
> 最后更新：2026-07-13  
> 项目代号：Shrimp（虾采购平台）

---

## 目录

1. [项目概述](#1-项目概述)
2. [产品定位与目标](#2-产品定位与目标)
3. [用户角色体系](#3-用户角色体系)
4. [功能需求](#4-功能需求)
   - [4.1 公共模块](#41-公共模块)
   - [4.2 客户前端](#42-客户前端)
   - [4.3 采购员端](#43-采购员端)
   - [4.4 商户后台](#44-商户后台)
   - [4.5 管理后台](#45-管理后台)
   - [4.6 支付系统](#46-支付系统)
   - [4.7 图片上传服务](#47-图片上传服务)
5. [系统架构](#5-系统架构)
6. [数据模型](#6-数据模型)
7. [接口规范](#7-接口规范)
8. [非功能需求](#8-非功能需求)
9. [技术栈](#9-技术栈)
10. [部署架构](#10-部署架构)

---

## 1. 项目概述

**Shrimp** 是一个面向生鲜/农产品领域的多角色在线采购与交易平台。平台连接了**普通消费者（采购需求方）**、**采购员（代购服务方）**、**商户（商品供应方）** 和**平台运营方（管理方）** 四方角色，提供从需求发布、代购接单、商品选购、订单履约到资金结算的全链路电商服务。

### 1.1 核心业务模式

```
消费者（发布采购意向） → 采购员（接单代购） → 商户（供应商品） → 物流配送 → 消费者收货
                               ↑                    ↑
                         平台运营管理 ←—— 平台方（监管/结算）
```

### 1.2 项目名称释义

"Shrimp"（虾）取自生鲜品类的典型代表，寓意平台立足生鲜农产品垂直领域，提供新鲜、高效、可信的采购交易服务。

---

## 2. 产品定位与目标

### 2.1 产品定位

一个**垂直领域的B2B2C采购交易平台**，专注于生鲜/农产品品类的代购与直购服务。

### 2.2 核心目标

| 目标 | 说明 |
|------|------|
| 降低采购门槛 | 普通消费者无需直接对接供应商，由专业采购员代为采购 |
| 提升供应链效率 | 通过意向聚合和采购员撮合，减少小批量采购成本 |
| 保障交易可信 | 平台提供资质审核、品质鉴定、资金托管（冻结/结算）等信任机制 |
| 多币种支持 | 支持人民币(CNY)、港币(HKD)、美元(USD)等多种货币交易 |

### 2.3 关键业务指标

- 用户注册量、活跃采购意向数
- 采购员接单率、订单履约率
- 商户入驻数、商品上架数
- 交易额、平台佣金收入
- 资金账户余额、充值/结算流水

---

## 3. 用户角色体系

系统定义了以下四种核心角色，同一用户可以同时拥有多种角色身份。

### 3.1 普通用户（客户/消费者）

| 属性 | 描述 |
|------|------|
| **身份标识** | 已注册的普通用户（默认角色） |
| **核心诉求** | 浏览商品、发布采购需求、查看订单状态 |
| **权限范围** | 公开商品查看、购物车、个人订单、发布/管理采购意向 |
| **前置条件** | 注册并登录 |

### 3.2 采购员（代购服务方）

| 属性 | 描述 |
|------|------|
| **身份标识** | 已通过采购资质审核的用户 |
| **核心诉求** | 接取采购意向、代客户选购商品、赚取佣金 |
| **权限范围** | 查看意向广场、接单、代客下单、管理采购资质 |
| **前置条件** | 申请采购资质并通过平台审批 |

### 3.3 商户（供货商家）

| 属性 | 描述 |
|------|------|
| **身份标识** | 拥有店铺（Merchant）的用户 |
| **核心诉求** | 管理商品库存、处理订单、查看营收数据 |
| **权限范围** | 商品CRUD、订单处理、店铺设置、数据统计、员工角色管理 |
| **前置条件** | 创建或被分配到店铺，且店铺状态为激活 |

### 3.4 平台管理员

| 属性 | 描述 |
|------|------|
| **身份标识** | 被授权为 PlatformAdmin 的用户 |
| **核心诉求** | 平台运营管理、商户/用户管理、资质审批、系统监控 |
| **权限细分** | 见下表 |
| **前置条件** | 超级管理员授权 |

#### 平台管理员级别

| 级别 | 常量值 | 说明 | 权限范围 |
|------|--------|------|----------|
| 超级管理员 | `super` | 拥有平台全部权限 | 所有模块所有动作（读/写/删/审批/导出） |
| 高级管理员 | `senior` | 管理多个业务模块 | 除"管理员自身管理"模块外的全部读写删审批导出 |
| 运营管理员 | `operator` | 日常运营操作 | 可读写和审批，不可删除或管理管理员 |
| 审计员 | `auditor` | 只读审计 | 仅可查看和导出数据 |

#### 平台管理模块

| 模块 | 标识 | 说明 |
|------|------|------|
| 仪表盘 | `dashboard` | 系统概况、统计数据 |
| 商户管理 | `merchant` | 商户入驻审核、信息管理 |
| 用户管理 | `user` | 用户信息查询与编辑 |
| 角色管理 | `role` | 平台角色与权限配置 |
| 采购资质 | `qualification` | 采购员资质审核 |
| 采购意向 | `intent` | 采购意向监管 |
| 商品管理 | `product` | 全平台商品监管 |
| 分类管理 | `category` | 商品分类维护 |
| 管理员管理 | `admin` | 平台管理员自身管理 |

---

## 4. 功能需求

### 4.1 公共模块

所有用户（含未登录游客）均可访问。

#### 4.1.1 用户注册

- **POST /api/v1/register**
- 支持用户名、手机号、密码注册
- 用户名唯一校验（3-50字符）
- 密码长度至少6位
- 注册成功返回用户基本信息

#### 4.1.2 用户登录

- **POST /api/v1/login**
- 支持用户名+密码登录
- 返回JWT双Token机制：`access_token`（短时效）+ `refresh_token`（长时效）
- 自动记录登录设备信息（操作系统、设备类型、浏览器、IP）
- 支持多设备同时登录

#### 4.1.3 商品浏览

- **GET /api/v1/product** — 商品列表（支持分页、分类筛选、关键词搜索、排序）
- **GET /api/v1/product/:id** — 商品详情
- 优先从Elasticsearch检索，ES不可用时自动降级到MySQL查询
- 支持按分类递归查询（含子分类）

#### 4.1.4 分类浏览

- **GET /api/v1/category** — 分类列表（树形结构，最多三级嵌套）
- **GET /api/v1/category/:id** — 分类详情

#### 4.1.5 用户资料

- **GET /api/v1/profile** — 获取当前用户完整资料（含角色、店铺、资金账户、采购资质、平台管理员信息）
- **GET /api/v1/session** — 获取登录设备列表

---

### 4.2 客户前端

面向普通消费者/采购需求方，需要登录认证。

#### 4.2.1 购物车

| 接口 | 方法 | 说明 |
|------|------|------|
| `/cart` | GET | 获取购物车列表 |
| `/cart/item` | POST | 添加商品到购物车（支持关联采购意向ID） |
| `/cart/item/:id` | PUT | 修改购物车项数量 |
| `/cart/item/:id` | DELETE | 删除购物车项 |
| `/cart` | DELETE | 清空购物车 |

购物车支持关联采购意向（IntentID），方便采购员为客户代购。

#### 4.2.2 订单管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/customer/order` | POST | 创建订单 |
| `/customer/order` | GET | 获取我的订单列表 |
| `/customer/order/:id` | GET | 订单详情 |
| `/customer/order/:id/cancel` | POST | 取消订单 |
| `/customer/order/:id/confirm` | POST | 确认收货 |
| `/customer/order/refund` | POST | 申请退款 |
| `/customer/order/refund` | GET | 退款记录查询 |

#### 4.2.3 物流查询

| 接口 | 方法 | 说明 |
|------|------|------|
| `/customer/shipping/track/:id` | GET | 物流追踪查询 |

#### 4.2.4 收货地址

| 接口 | 方法 | 说明 |
|------|------|------|
| `/customer/address` | GET | 地址列表 |
| `/customer/address` | POST | 新增地址 |

#### 4.2.5 消息通知

| 接口 | 方法 | 说明 |
|------|------|------|
| `/customer/message` | GET | 获取消息列表 |
| `/customer/message/:id/read` | POST | 标记消息已读 |
| `/customer/message/:id` | DELETE | 删除消息 |

#### 4.2.6 采购意向发布

| 接口 | 方法 | 说明 |
|------|------|------|
| `/customer/intent` | POST | 发布采购意向 |
| `/customer/intent/me` | GET | 我的采购意向列表 |
| `/customer/intent/:id/cancel` | POST | 取消采购意向 |

采购意向包含：标题、描述、采购货品清单、佣金金额、预估总金额、品质要求、位置信息、收货地址等。

---

### 4.3 采购员端

面向通过资质审核的采购员，提供代购接单能力。

#### 4.3.1 采购资质

| 接口 | 方法 | 说明 |
|------|------|------|
| `/procurement/qualification/apply` | POST | 申请采购资质（含等级和等级名称） |
| `/procurement/qualification/me` | GET | 查询本人的采购资质状态 |

资质状态流转：`pending`（待审批）→ `active`（已激活）/ `inactive`（已停用）

#### 4.3.2 意向广场

| 接口 | 方法 | 说明 |
|------|------|------|
| `/procurement/intent/square` | GET | 公开采购意向列表（分页） |
| `/procurement/intent/:id` | GET | 意向详情 |
| `/procurement/intent/:id/accept` | POST | 接单（接受采购意向） |
| `/procurement/intent/mine` | GET | 我接的采购意向列表 |
| `/procurement/intent/:id/order` | GET | 意向关联的订单 |

意向状态流转：`pending`（待接单）→ `accepted`（已接单）→ `processing`（处理中）→ `completed`（已完成）/ `cancelled`（已取消）

---

### 4.4 商户后台

面向供货商/店铺管理者，提供完整的商品和订单管理能力。所有接口需经过认证 + 商户解析 + 权限验证三层中间件。

#### 4.4.1 商品管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/product` | GET | 商品列表 |
| `/merchant/product/:id` | GET | 商品详情 |
| `/merchant/product` | POST | 新增商品 |
| `/merchant/product/:id` | PUT | 更新商品 |
| `/merchant/product/:id` | DELETE | 删除商品 |

商品属性包含：名称、描述、价格（分单位）、库存、分类、品牌、SKU、计量单位、图片（JSON数组）、状态。

#### 4.4.2 订单管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/order` | GET | 订单列表 |
| `/merchant/order/:id` | GET | 订单详情 |
| `/merchant/order/:id/status` | PUT | 更新订单状态 |
| `/merchant/order/:id/ship` | PUT | 发货操作（含物流信息） |
| `/merchant/order/:id/cancel` | PUT | 取消订单 |
| `/merchant/order/:id/item` | GET | 订单商品明细 |
| `/merchant/order/statistic` | GET | 订单统计 |

#### 4.4.3 库存管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/inventory` | GET | 库存列表 |
| `/merchant/inventory/:productId` | GET | 特定商品库存 |
| `/merchant/inventory/:productId` | PUT | 更新库存 |
| `/merchant/inventory/low-stock` | GET | 低库存预警列表 |
| `/merchant/inventory/alert` | GET | 库存警报设置 |

#### 4.4.4 分类管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/category` | GET | 分类列表 |
| `/merchant/category/:id` | GET | 分类详情 |
| `/merchant/category` | POST | 新增分类 |
| `/merchant/category/:id` | PUT | 更新分类 |
| `/merchant/category/:id` | DELETE | 删除分类 |
| `/merchant/category/tree` | GET | 分类树 |

#### 4.4.5 店铺管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/merchant` | GET | 查看店铺信息 |
| `/merchant/merchant` | PUT | 更新店铺信息 |
| `/merchant/merchant` | POST | 创建店铺 |

店铺属性包含：名称、Logo、横幅、描述、联系电话、地址、省/市/区、抽成比例。

#### 4.4.6 数据统计

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/analytic/overview` | GET | 概览统计 |
| `/merchant/analytic/sale` | GET | 销售分析 |
| `/merchant/analytic/revenue` | GET | 收入分析 |
| `/merchant/analytic/customer` | GET | 客户分析 |
| `/merchant/analytic/product` | GET | 商品分析 |

#### 4.4.7 角色权限管理

| 接口 | 方法 | 说明 |
|------|------|------|
| `/merchant/role` | GET | 角色列表 |
| `/merchant/role/:id` | GET | 角色详情 |
| `/merchant/role` | POST | 新建角色 |
| `/merchant/role/:id` | PUT | 更新角色 |
| `/merchant/role/:id` | DELETE | 删除角色 |

商户角色权限模型：基于模型（Model）+ HTTP方法（Method）的细粒度权限控制，各角色关联权限集合（Permission），Admin角色拥有全部权限。

---

### 4.5 管理后台

面向平台运营管理团队，提供全平台监管能力，基于模块+动作的细粒度权限控制。

#### 4.5.1 仪表盘

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/dashboard` | GET | dashboard:read | 管理后台首页统计 |
| `/admin/stats` | GET | dashboard:read | 系统统计数据 |

#### 4.5.2 商户管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/merchant` | GET | merchant:read | 商户列表 |
| `/admin/merchant/:id` | GET | merchant:read | 商户详情 |
| `/admin/merchant` | POST | merchant:write | 创建商户 |
| `/admin/merchant/:id` | PUT | merchant:write | 更新商户 |
| `/admin/merchant/:id` | DELETE | merchant:delete | 删除商户 |

#### 4.5.3 用户管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/user` | GET | user:read | 用户列表 |
| `/admin/user/:id` | GET | user:read | 用户详情 |
| `/admin/user/:id` | PUT | user:write | 更新用户信息 |

#### 4.5.4 角色管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/role` | GET | role:read | 角色列表 |
| `/admin/role/:id` | GET | role:read | 角色详情 |
| `/admin/role` | POST | role:write | 创建角色 |
| `/admin/role/:id` | PUT | role:write | 更新角色 |
| `/admin/role/:id` | DELETE | role:delete | 删除角色 |

#### 4.5.5 采购资质管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/qualification` | GET | qualification:read | 资质列表 |
| `/admin/qualification/:id` | GET | qualification:read | 资质详情 |
| `/admin/qualification` | POST | qualification:write | 创建资质 |
| `/admin/qualification/:id` | PUT | qualification:write | 更新资质 |
| `/admin/qualification/:id` | DELETE | qualification:delete | 删除资质 |
| `/admin/qualification/:id/approve` | POST | qualification:approve | 审批资质 |

#### 4.5.6 采购意向管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/intent` | GET | intent:read | 采购意向列表 |
| `/admin/intent/:id` | GET | intent:read | 采购意向详情 |

#### 4.5.7 商品管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/product` | GET | product:read | 商品列表 |
| `/admin/product/:id` | GET | product:read | 商品详情 |
| `/admin/product` | POST | product:write | 创建商品 |
| `/admin/product/:id` | PUT | product:write | 更新商品 |
| `/admin/product/:id` | DELETE | product:delete | 删除商品 |

#### 4.5.8 分类管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/category` | GET | category:read | 分类列表 |
| `/admin/category/:id` | GET | category:read | 分类详情 |
| `/admin/category` | POST | category:write | 创建分类 |
| `/admin/category/:id` | PUT | category:write | 更新分类 |
| `/admin/category/:id` | DELETE | category:delete | 删除分类 |

#### 4.5.9 平台管理员管理

| 接口 | 方法 | 权限 | 说明 |
|------|------|------|------|
| `/admin/admin` | GET | admin:read | 管理员列表 |
| `/admin/admin/:id` | GET | admin:read | 管理员详情 |
| `/admin/admin` | POST | admin:write | 创建管理员 |
| `/admin/admin/:id` | PUT | admin:write | 更新管理员 |
| `/admin/admin/:id` | DELETE | admin:delete | 删除管理员 |
| `/admin/admin/:id/permissions` | PUT | admin:write | 更新管理员权限 |

---

### 4.6 支付系统

提供支付宝和微信支付两种充值渠道（当前仅支持充值，暂不支持直接支付订单）。支付系统对接第三方支付网关，处理异步通知和支付结果回调。

#### 4.6.1 支付宝

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/payment/alipay/deposit` | POST | 创建支付宝充值订单 |
| `/api/v1/payment/alipay/notify` | POST | 支付宝异步通知回调 |
| `/api/v1/payment/alipay/return` | GET | 支付宝同步跳转回页 |

#### 4.6.2 微信支付

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/v1/payment/wechat/deposit` | POST | 创建微信充值订单 |
| `/api/v1/payment/wechat/notify` | POST | 微信异步通知回调 |
| `/api/v1/payment/wechat/return` | GET | 微信同步跳转回页 |

#### 4.6.3 资金账户体系

| 概念 | 说明 |
|------|------|
| **Account** | 每用户每币种一个资金账户，记录可用余额和冻结余额 |
| **可用余额** | 可自由支配的资金 |
| **冻结余额** | 订单进行中暂扣的预付款，不可动用 |
| **乐观锁** | 使用版本号（Version）控制并发更新 |
| **资金变动日志** | 所有资金变动记录在 AccountVersion 表中，支持审计追溯 |
| **幂等防重** | (source_type, source_id, account_id) 联合唯一约束防止重复入账 |
| **行级锁** | `SELECT ... FOR UPDATE` 保证事务安全 |

资金操作类型：
- `FunCredit (1)` — 增加可用余额（充值入账）
- `FunDebit (-1)` — 减少可用余额（扣款）
- `FunFreeze (0)` — 冻结：available → frozen
- `FunUnfreeze (2)` — 解冻：frozen → available
- `Settle` — 从冻结余额扣除（确认收入）

#### 4.6.4 充值模型

| 字段 | 说明 |
|------|------|
| Deposit | 充值记录（pending → paid → canceled/failed） |
| Income | 资金到账记录（pending → confirmed → refunded） |
| Refund | 退款记录（pending → processing → completed/failed） |
| PayMethod | 支付方式：alipay / wechat / bank / point |

---

### 4.7 图片上传服务

独立部署的图片上传微服务（端口 8091），全平台统一使用。

| 接口 | 方法 | 说明 |
|------|------|------|
| `/image/upload` | POST | 上传图片（含安全校验：格式、大小、内容检测） |
| `/image/delete` | DELETE | 删除图片 |
| `/health` | GET | 健康检查 |

支持的图片格式：JPEG、PNG、GIF、WebP  
上传限制：最大 10MB  
存储后端：支持本地存储和阿里云OSS，通过配置文件切换

---

## 5. 系统架构

### 5.1 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                       前端 (shrimp-ex)                       │
│                   Vue 3 SPA (端口 5173)                      │
│             路由前缀 /ex/ → 反向代理 /api → 后端              │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / JSON
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  Gin HTTP 服务 (端口 8090)                    │
│                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────┐  │
│  │ 公共API  │ │ 客户API  │ │ 商户API  │ │  管理后台API   │  │
│  │          │ │          │ │          │ │               │  │
│  │ 商品/分类 │ │ 购物车/  │ │ 商品/订单 │ │ 商户/用户/    │  │
│  │ 注册/登录 │ │ 订单/地址 │ │ 库存/分类 │ │ 资质/意向/    │  │
│  │          │ │ 意向/消息 │ │ 统计/角色 │ │ 商品/管理员   │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └───────┬───────┘  │
│       │             │            │                │          │
│       └──────┬──────┴────────────┴────────────────┘          │
│              ▼                                               │
│     ┌────────────────────┐                                   │
│     │   Workshop 层      │  ← 业务逻辑层（Domain Service）     │
│     └────────┬───────────┘                                   │
│              ▼                                               │
│     ┌────────────────────┐                                   │
│     │   Infrastructure   │  ← 基础设施（MySQL/Redis/ES/OSS） │
│     └────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────────┐
          ▼                ▼                    ▼
     ┌────────┐      ┌────────┐         ┌──────────────┐
     │ MySQL  │      │ Redis  │         │Elasticsearch │
     │ 主数据 │      │ 缓存   │         │ 搜索引擎     │
     └────────┘      └────────┘         └──────────────┘
```

### 5.2 后端分层架构

```
service/api/main.go          ← 入口：初始化 + HTTP服务启动
  │
  ├── middleware/             ← HTTP中间件层
  │   ├── auth.go            ← JWT认证
  │   ├── admin.go           ← 平台管理员权限校验
  │   └── merchant.go        ← 商户解析 + 模型权限校验
  │
  ├── service/api/v1/        ← Controller/Handler层
  │   ├── user_handler.go    ← 用户相关接口
  │   ├── product_handler.go ← 商品接口
  │   ├── category_handler.go← 分类接口
  │   ├── customer/          ← 客户前端接口
  │   ├── merchant/          ← 商户后台接口
  │   ├── admin/             ← 管理后台接口
  │   ├── procurement/       ← 采购员接口
  │   └── payment/           ← 支付接口
  │
  ├── internal/workshop/     ← 业务逻辑层（Domain Service）
  │   ├── user_workshop.go   ← 用户业务（注册/登录/资料）
  │   ├── product_workshop.go← 商品业务
  │   ├── customer/          ← 客户业务
  │   ├── merchant/          ← 商户业务
  │   ├── procurement/       ← 采购业务
  │   ├── payment/           ← 支付业务
  │   └── admin/             ← 管理后台业务
  │
  ├── internal/account/      ← 资金账户领域
  │   └── account.go         ← 余额操作（增/减/冻结/解冻/结算）
  │
  ├── internal/cache/        ← 缓存层
  │   ├── role.go            ← 角色缓存
  │   ├── merchant.go        ← 商户缓存
  │   └── platform_admin.go  ← 平台管理员缓存
  │
  ├── model/                 ← 数据模型层（GORM映射）
  │
  ├── pkg/                   ← 基础设施包
  │   ├── mysql/             ← MySQL连接
  │   ├── redis/             ← Redis连接
  │   ├── elasticsearch/     ← ES搜索
  │   ├── config/            ← 配置加载
  │   ├── i18n/              ← 国际化
  │   └── storage/           ← 文件存储
  │
  ├── util/                  ← 工具包
  │   ├── jwt.go             ← JWT生成/验证
  │   ├── password.go        ← 密码哈希/验证
  │   └── useragent.go       ← UA解析
  │
  └── config/                ← 配置文件
      ├── app.yml            ← 应用配置
      └── i18n/              ← 国际化翻译文件
```

### 5.3 前端架构

```
shrimp-ex/src/
  ├── main.js                ← 入口：创建Vue应用，挂载Pinia/Router
  ├── App.vue                ← 根组件（NavBar + router-view）
  ├── style.css              ← 全局样式（CSS变量、工具类）
  │
  ├── router/index.js        ← 路由配置
  │   ├── 公共路由：/login, /register, /products
  │   ├── 客户路由：/cart, /orders, /intent/*
  │   ├── 采购员路由：/procurement/*
  │   ├── 商户路由：/merchant/*
  │   └── 管理后台路由：/admin/*
  │
  ├── stores/                ← Pinia状态管理
  │   ├── auth.js            ← 认证状态（登录/登出/角色判断）
  │   ├── cart.js            ← 购物车状态
  │   ├── admin.js           ← 管理后台状态
  │   ├── merchant.js        ← 商户后台状态
  │   └── procurement.js     ← 采购员状态
  │
  ├── utils/api.js           ← Axios实例（拦截器、Token注入）
  │
  ├── components/            ← 通用组件
  │   ├── NavBar.vue         ← 导航栏（根据角色动态显示菜单）
  │   └── ProductCard.vue    ← 商品卡片
  │
  └── views/                 ← 页面组件
      ├── customer/          ← 客户前端页面
      │   ├── Login.vue / Register.vue
      │   ├── ProductList.vue / ProductDetail.vue
      │   ├── Cart.vue / Checkout.vue / Orders.vue
      │   ├── PublishIntent.vue / CustomerIntents.vue
      │
      ├── procurement/       ← 采购员页面
      │   ├── ProcurementDashboard.vue
      │   ├── IntentSquare.vue / IntentDetail.vue
      │   ├── MyIntents.vue / Qualification.vue
      │
      ├── merchant/          ← 商户页面
      │   ├── MerchantDashboard.vue
      │   ├── ProductManage.vue / OrderManage.vue
      │
      └── admin/             ← 管理后台页面
          ├── AdminDashboard.vue
          ├── MerchantManage.vue / UserManage.vue
          ├── QualificationManage.vue / IntentManage.vue
```

---

## 6. 数据模型

### 6.1 核心实体关系

```
User
  ├── Roles (多对多: user_role 关联表)
  ├── CartItems (一对多: 购物车项)
  ├── Orders (一对多: 订单)
  ├── Favorites (一对多: 收藏)
  ├── LoginRecords (一对多: 登录记录)
  ├── Messages (一对多: 消息)
  ├── ShippingAddresses (一对多: 收货地址)
  ├── ProcurementQualification (一对一: 采购资质)
  ├── ProcurementIntent (一对多: 采购意向)
  ├── Merchant (一对多: 拥有的店铺)
  ├── PlatformAdmin (一对一: 平台管理员身份)
  └── Accounts (一对多: 资金账户)

Merchant
  ├── Staff (一对多: 员工)
  ├── Roles (一对多: 店铺角色)
  └── Products (一对多: 商品)

Product
  ├── Category (多对一: 分类)
  ├── Currency (多对一: 货币)
  └── Merchant (多对一: 店铺)

Order
  ├── OrderItems (一对多: 订单明细)
  ├── User (多对一: 下单用户)
  ├── Procurement (多对一: 采购员)
  ├── Merchant (多对一: 店铺)
  └── Intent (多对一: 采购意向)

Account + AccountVersion (资金账户 + 变动日志)
Payment: Deposit → Income → Refund
```

### 6.2 主要数据表

| 表名 | 说明 | 关键字段 |
|------|------|----------|
| `user` | 用户表 | username(唯一), phone(唯一), password(bcrypt), avatar, status |
| `role` | 角色表 | name, merchant_id, admin(是否后台权限) |
| `permission` | 权限表 | model(模型名), method(HTTP方法) |
| `role_permission` | 角色权限关联 | role_id, permission_id |
| `user_role` | 用户角色关联 | user_id, role_id, scope, merchant_id |
| `merchant` | 商户/店铺表 | name, owner_id, logo, commission(抽成比例) |
| `merchant_staff` | 店铺员工表 | merchant_id, user_id, role_id, position |
| `product` | 商品表 | merchant_id, name, price(分), stock, category_id, sku, unit, images(JSON) |
| `category` | 分类表 | name, parent_id(自关联树形结构) |
| `cart_item` | 购物车项 | user_id, product_id, quantity, intent_id |
| `order` | 订单表 | intent_id, procurement_id, merchant_id, user_id, order_number, total_amount(分), commission_amt, status, account_pay_status |
| `order_item` | 订单项 | order_id, product_id, product_name, unit_price(分), quantity, subtotal |
| `favorite` | 收藏表 | user_id, product_id(联合唯一) |
| `currency` | 货币表 | code(CNY/HKD/USD), name, symbol, rate(万分比) |
| `account` | 资金账户 | user_id, currency_id, available, frozen, version(乐观锁) |
| `account_version` | 资金变动日志 | account_id, source_type, source_id, 变动前后金额 |
| `deposit` | 充值记录 | user_id, amount, pay_method, trade_no, status |
| `income` | 到账记录 | user_id, deposit_id, amount, payment_method, status |
| `refund` | 退款记录 | income_id, order_id, amount, reason, status |
| `procurement_qualification` | 采购资质 | user_id, level, level_name, total_amount, status |
| `procurement_intent` | 采购意向 | user_id, procurement_id, title, description, products, commission_amt, estimated_amount, status |
| `shipping_address` | 收货地址 | user_id, name, phone, province/city/district, detail, is_default |
| `login_record` | 登录记录 | user_id, token, device_id/OS/type, browser, ip, login_at |
| `user_message` | 用户消息 | user_id, type, title, content, is_read |
| `image` | 图片记录 | user_id, file_key, file_size, image_type, storage_type |
| `platform_admin` | 平台管理员 | user_id, level(super/senior/operator/auditor), status |
| `platform_admin_permission` | 管理员权限 | admin_id, module, action |

### 6.3 状态机

**订单状态**:
```
pending → confirmed → packaged → verified → picked → delivered → completed
     ↓         ↓
  cancelled   cancelled
```

**采购意向状态**:
```
pending → accepted → processing → completed
     ↓                          cancelled
```

**采购资质状态**:
```
pending → active → inactive
```

**充值状态**:
```
pending → paid → completed
     ↓      ↓
  canceled  failed
```

**资金到账状态**:
```
pending → confirmed
     ↓
  refunded
```

---

## 7. 接口规范

### 7.1 统一响应格式

所有接口使用统一的 JSON 响应结构：

```json
{
  "Head": {
    "Code": "1000",
    "Msg": "Success"
  },
  "Body": { ... },
  "Pagination": { ... }
}
```

### 7.2 字段命名规范

API 响应中的 JSON 字段统一使用 **PascalCase（首字母大写的驼峰命名法）**：
- `UserID`、`OrderNumber`、`CreatedAt`、`TotalAmount`

### 7.3 通用错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 1000 | 200 | 成功 |
| 1001 | 400 | 请求参数错误 |
| 1002 | 401 | 未登录/令牌无效 |
| 1003 | 403 | 无权限 |
| 1004 | 404 | 资源不存在 |
| 1005 | 500 | 服务器内部错误 |
| 1006 | 400 | 数据格式不正确 |
| 1007 | 409 | 数据已存在 |

### 7.4 认证错误码

| 错误码 | HTTP状态 | 说明 |
|--------|----------|------|
| 1401 | 401 | 用户名或密码错误 |
| 1402 | 403 | 账户已被禁用 |
| 1403 | 401 | 无效的令牌 |
| 1404 | 401 | 令牌已过期 |
| 1405 | 401 | 令牌类型不正确 |
| 1406 | 401 | 无效的刷新令牌 |
| 1407 | 403 | 无权限访问此资源 |

### 7.5 分页格式

```json
{
  "Per": 10,
  "Count": 56,
  "Page": 6,
  "Current": 1,
  "Next": 2,
  "Previous": 0,
  "Order": "id:desc"
}
```

### 7.6 认证方式

```
Authorization: Bearer <access_token>
```

### 7.7 国际化支持

通过 `Accept-Language` 请求头控制响应消息语言：
- `zh-CN`（默认）：中文
- `en`：英文

---

## 8. 非功能需求

### 8.1 性能需求

| 指标 | 要求 |
|------|------|
| 商品列表查询 | 95% 请求响应时间 < 500ms（ES缓存命中时 < 200ms） |
| 订单创建 | 95% 请求响应时间 < 1s |
| 登录/注册 | 95% 请求响应时间 < 1s |
| 并发支持 | 支持 100+ 并发请求（受限流配置控制，默认 100 RPS，突发 50） |
| 数据库连接池 | 最大连接数 100，空闲连接数 10，连接最长存活 300s |

### 8.2 安全需求

| 需求 | 说明 |
|------|------|
| 密码安全 | 使用 bcrypt 哈希存储密码，成本因子 ≥ 10 |
| JWT 令牌 | 双 Token 机制，access 短时效 + refresh 长时效 |
| 行级锁 | 资金操作使用 `SELECT ... FOR UPDATE` 保证并发安全 |
| 幂等防重 | (source_type, source_id, account_id) 联合唯一约束防止资金重复入账 |
| CORS | 配置白名单跨域（开发环境 localhost:3000） |
| 图片安全 | 上传格式白名单（JPEG/PNG/GIF/WebP），大小限制 10MB |
| 权限校验 | 三层中间件：Auth → MerchantResolver → RequireModelPermission |

### 8.3 可用性需求

| 需求 | 说明 |
|------|------|
| 降级策略 | Redis 不可用时购物车降级 MySQL；ES 不可用时商品搜索降级 MySQL |
| 健康检查 | `/health` 端点返回服务状态 |
| Prometheus | 可选的监控指标暴露端点 `/metrics` |
| Docker化 | 支持 Docker Compose 一键部署 |

### 8.4 可维护性需求

| 需求 | 说明 |
|------|------|
| 代码分层 | Controller → Workshop(Service) → Repository 清晰分层 |
| 配置分离 | 应用配置通过 `config/app.yml` 管理，支持环境变量覆盖 |
| 日志系统 | 支持 debug/info/warn/error 级别，text/json 格式，stdout/file 输出 |
| 国际化 | 错误消息支持中英文，通过 i18n 配置管理 |
| API文档 | 使用 Swaggo 自动生成 Swagger 文档 |
| 数据库迁移 | 使用 GORM AutoMigrate 自动维护表结构 |

### 8.5 兼容性需求

- 前端支持现代浏览器（Chrome、Firefox、Safari、Edge 最近两个大版本）
- API 版本路径前缀 `/api/v1/`，支持后续版本演进

---

## 9. 技术栈

### 9.1 后端技术栈

| 技术 | 用途 | 版本 |
|------|------|------|
| Go | 编程语言 | 1.25 |
| Gin | HTTP Web 框架 | v1.10.0 |
| GORM | ORM 框架 | v1.30.0 |
| MySQL | 关系型数据库 | 8.0 |
| Redis | 缓存（可选） | 7.x |
| Elasticsearch | 搜索引擎（可选） | 8.x |
| JWT | 身份认证 | golang-jwt |
| Aliyun OSS | 对象存储 | 阿里云 SDK |
| bcrypt | 密码哈希 | golang.org/x/crypto |
| Swaggo | API 文档生成 | v1.16.6 |
| i18n | 国际化 | 自研 |

### 9.2 前端技术栈

| 技术 | 用途 | 版本 |
|------|------|------|
| Vue 3 | 前端框架 | v3.5.32 |
| Pinia | 状态管理 | v3.0.4 |
| Vue Router | 前端路由 | v4.6.4 |
| Axios | HTTP 客户端 | v1.16.0 |
| Vite | 构建工具 | v8.0.10 |

### 9.3 基础设施

| 技术 | 用途 |
|------|------|
| Docker | 容器化部署 |
| Docker Compose | 多容器编排 |
| Alibaba Cloud | 云基础设施 |
| Prometheus | 监控指标（可选） |

---

## 10. 部署架构

### 10.1 服务组件

| 组件 | 端口 | 说明 |
|------|------|------|
| API 服务 (api) | 8090 | 主业务 API，提供全部 REST 接口 |
| 图片服务 (image) | 8091 | 图片上传与存储管理 |
| MySQL | 3306 | 数据库主存储 |
| Redis | 6379 | 缓存（可选） |
| Elasticsearch | 9200 | 搜索引擎（可选） |
| 前端 (shrimp-ex) | 5173(dev) | Vue 3 SPA（生产构建后由 API 服务托管） |

### 10.2 前端部署策略

- 开发环境：Vite Dev Server（端口 5173），API 请求反向代理至 8090
- 生产环境：Vite 构建输出到 `shrimp/public/dist/`，由 Gin 路由 `/ex/` 前缀提供静态文件服务，SPA 路由 fallback 统一返回 `index.html`

### 10.3 容器化部署

```yaml
服务编排:
  - mysql (8.0, 数据持久化卷)
  - redis (7-alpine)
  - api (Go编译二进制, Dockerfile构建)
  - image (独立微服务, Dockerfile构建)
```

---

## 附录

### A. 关键业务术语表

| 术语 | 说明 |
|------|------|
| 采购意向 | 消费者发布的采购需求，包含货品清单、佣金、品质要求等 |
| 意向广场 | 采购员可浏览和接取的公开采购需求列表 |
| 采购资质 | 采购员需通过平台审批才能接单的资格认证 |
| 预付款冻结 | 下单时将消费者账户的可用余额冻结，保障交易安全 |
| 佣金 | 消费者支付给采购员的代购服务费 |
| 店铺抽成 | 平台从商户交易额中抽取的佣金比例 |
| 资金变动日志 | 记录每一笔资金变动的审计日志，支持追溯 |

### B. 数据库命名规范

- 表名使用单数形式（如 `user`、`order`、`product`）
- 字段使用 PascalCase（如 `UserID`、`CreatedAt`）
- 唯一索引前缀 `uidx_`，普通索引前缀 `idx_`
- 外键字段使用关联表名 + ID（如 `MerchantID`、`CategoryID`）
- 金额字段统一使用 `int64` 类型，单位为**分**
- 状态字段使用 `string` 类型，长度 20
- 所有表包含 `CreatedAt`、`UpdatedAt` 时间字段
- 逻辑删除字段使用 `DeletedAt`（gorm.DeletedAt 类型）

### C. API 路由前缀汇总

| 前缀 | 认证 | 说明 |
|------|------|------|
| `/api/v1/product` | 否 | 公共商品浏览 |
| `/api/v1/category` | 否 | 公共分类浏览 |
| `/api/v1/register` | 否 | 用户注册 |
| `/api/v1/login` | 否 | 用户登录 |
| `/api/v1/profile` | 是 | 用户资料 |
| `/api/v1/session` | 是 | 登录会话 |
| `/api/v1/cart` | 是 | 购物车 |
| `/api/v1/customer/*` | 是 | 客户前端 |
| `/api/v1/procurement/*` | 是 | 采购员 |
| `/api/v1/merchant/*` | 是 | 商户后台 |
| `/api/v1/admin/*` | 是 | 管理后台 |
| `/api/v1/payment/*` | 否 | 支付 |
| `/image/*` | 否 | 图片上传服务 |
| `/health` | 否 | 健康检查 |
