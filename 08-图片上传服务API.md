# 图片上传服务 API

图片上传服务是一个独立的微服务节点（`service/image`），为全平台提供统一、安全的图片上传、校验和存储能力。

## 基础信息

| 项目 | 值 |
|------|-----|
| 服务地址 | `http://localhost:8091` |
| 数据格式 | JSON |
| 字符编码 | UTF-8 |

---

## 1. 上传图片

```
POST /image/upload
```

上传一张图片，经过安全校验（魔术字节验证、解码重编码去元数据、MIME 双校验、文件大小和尺寸限制）后，存储到 OSS 并记录到 `image` 表。

### 请求格式

`multipart/form-data`

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `file` | file | 是 | 图片文件，支持 jpg/png/gif/webp |

### Curl 示例

```bash
curl -X POST "http://localhost:8091/image/upload" \
  -F "file=@/path/to/logo.jpg"
```

### 响应示例

```json
{
  "url": "https://oss-cn-heyuan.aliyuncs.com/images/2026/06/28/a1b2c3d4e5f6...jpg",
  "key": "images/2026/06/28/a1b2c3d4e5f6...jpg",
  "size": 102456,
  "ext": "a1b2c3d4e5f6...jpg"
}
```

### 响应字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `url` | string | 图片可访问 URL（**非永久有效**，长期使用需通过 `oss.GetObjectURL` 生成临时签名链接） |
| `key` | string | OSS 存储键，用于后续删除或生成签名 URL |
| `size` | int64 | 文件大小（字节） |
| `ext` | string | 安全存储文件名（UUID 重命名） |

### 错误响应

```json
{
  "code": 400,
  "message": "图片校验失败: 文件大小 20971520 超过限制 10485760 bytes"
}
```

---

## 2. 删除图片

```
DELETE /image/delete
```

按 OSS key 删除已上传的图片。

### 请求体

```json
{
  "key": "images/2026/06/28/a1b2c3d4e5f6...jpg"
}
```

### 响应示例

```json
{
  "message": "删除成功"
}
```

---

## 3. 健康检查

```
GET /health
```

### 响应示例

```json
{
  "status": "healthy",
  "service": "image-upload",
  "timestamp": "2026-06-28T12:00:00+08:00"
}
```

---

## 安全校验说明

图片上传经过以下多层安全防护：

| 层级 | 校验内容 | 说明 |
|------|----------|------|
| ① | 文件大小 | 默认 ≤ 10MB，可配置 |
| ② | 魔术字节 | 检查文件头签名，JPEG:`FF D8 FF`、PNG:`89 50 4E 47`、GIF:`47 49 46`、WebP:`52 49 46 46` |
| ③ | MIME 双校验 | `Content-Type` header 与魔术字节必须一致 |
| ④ | 解码重编码 | JPEG/PNG/GIF 解码后重新编码，剥离 EXIF、注释、嵌入代码 |
| ⑤ | 维度限制 | 默认 ≤ 4096×4096px，可配置 |
| ⑥ | 文件名安全 | UUID 重命名，保留扩展名，防路径穿越 |

---

## 数据库记录

上传成功后会在 `image` 表中记录一条元数据，字段如下：

| 字段 | 类型 | 说明 |
|------|------|------|
| `ID` | uint | 主键 |
| `UserID` | uint | 上传用户 ID |
| `OriginalName` | string | 原始文件名 |
| `StoredName` | string | 安全存储文件名 |
| `FileKey` | string | OSS 存储键（已建索引） |
| `FileSize` | int64 | 文件大小 |
| `ImageType` | string | 图片格式（jpeg/png/gif/webp） |
| `Width` | int | 图片宽度 |
| `Height` | int | 图片高度 |
| `ContentType` | string | MIME 类型 |
| `StorageType` | string | 存储后端（oss） |
| `Source` | string | 来源标识，如 `merchant_logo` |
| `CreatedAt` | timestamp | 创建时间 |
| `DeletedAt` | timestamp | 软删除时间 |

> URL 不持久化存储，需要时通过 `FileKey` 调用 OSS SDK 的 `GetObjectURL` 方法生成带签名的临时下载链接。
