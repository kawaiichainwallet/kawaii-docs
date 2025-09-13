# API 接口文档

## 基本信息

| 属性 | 值 |
|------|---|
| **Base URL** | `https://api.wallet-project.com` |
| **协议** | HTTPS |
| **数据格式** | JSON |
| **认证方式** | JWT Token |
| **API版本** | v1 |

## 认证机制

### JWT Token 获取

所有需要认证的接口都需要在请求头中携带JWT Token：

```http
Authorization: Bearer <your_jwt_token>
```

### Token 刷新

Token 有效期为24小时，可通过刷新接口延长有效期。

## 通用响应格式

### 成功响应
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    // 具体数据
  },
  "timestamp": 1693123456789
}
```

### 错误响应
```json
{
  "code": 400,
  "message": "参数错误",
  "data": null,
  "timestamp": 1693123456789,
  "error_details": {
    "field": "email",
    "reason": "邮箱格式不正确"
  }
}
```

### 状态码说明

| 状态码 | 说明 |
|--------|------|
| 200 | 请求成功 |
| 400 | 请求参数错误 |
| 401 | 未授权访问 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

## 用户认证 API

### 用户注册

**接口地址**
```
POST /api/v1/auth/register
```

**请求参数**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "securePassword123",
  "confirmPassword": "securePassword123",
  "verificationCode": "123456"
}
```

**响应示例**
```json
{
  "code": 200,
  "message": "注册成功",
  "data": {
    "userId": "1001",
    "username": "john_doe",
    "email": "john@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

### 用户登录

**接口地址**
```
POST /api/v1/auth/login
```

**请求参数**
```json
{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**响应示例**
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJl...",
    "expiresIn": 86400,
    "user": {
      "id": "1001",
      "username": "john_doe",
      "email": "john@example.com"
    }
  }
}
```

### Token 刷新

**接口地址**
```
POST /api/v1/auth/refresh
```

**请求参数**
```json
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl..."
}
```

## 钱包管理 API

### 创建钱包

**接口地址**
```
POST /api/v1/wallets
```

**请求头**
```
Authorization: Bearer <jwt_token>
```

**请求参数**
```json
{
  "name": "我的以太坊钱包",
  "chainType": "ethereum",
  "password": "walletPassword123"
}
```

**响应示例**
```json
{
  "code": 200,
  "message": "钱包创建成功",
  "data": {
    "walletId": "w_1001",
    "name": "我的以太坊钱包",
    "address": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
    "chainType": "ethereum",
    "mnemonic": "abandon ability able about above absent absorb abstract absurd abuse access",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

### 导入钱包

**接口地址**
```
POST /api/v1/wallets/import
```

**请求参数**
```json
{
  "name": "导入的钱包",
  "importType": "mnemonic",
  "importData": "abandon ability able about above absent absorb abstract absurd abuse access",
  "password": "walletPassword123"
}
```

**支持的导入类型**
- `mnemonic` - 助记词
- `privateKey` - 私钥
- `keystore` - Keystore文件

### 获取钱包列表

**接口地址**
```
GET /api/v1/wallets
```

**查询参数**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| page | int | 否 | 页码，默认1 |
| size | int | 否 | 每页数量，默认10 |
| chainType | string | 否 | 链类型筛选 |

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "wallets": [
      {
        "walletId": "w_1001",
        "name": "我的以太坊钱包",
        "address": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
        "chainType": "ethereum",
        "balance": "1.234567890123456789",
        "isActive": true,
        "createdAt": "2024-01-15T10:30:00Z"
      }
    ],
    "total": 5,
    "page": 1,
    "size": 10
  }
}
```

### 获取钱包余额

**接口地址**
```
GET /api/v1/wallets/{walletId}/balance
```

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "walletId": "w_1001",
    "balances": [
      {
        "token": "ETH",
        "balance": "1.234567890123456789",
        "usdValue": "2468.91"
      },
      {
        "token": "USDT",
        "balance": "1000.000000",
        "usdValue": "1000.00",
        "contractAddress": "0xdAC17F958D2ee523a2206206994597C13D831ec7"
      }
    ]
  }
}
```

## 交易管理 API

### 发起转账

**接口地址**
```
POST /api/v1/transactions/transfer
```

**请求参数**
```json
{
  "fromWalletId": "w_1001",
  "toAddress": "0x8ba1f109551bD432803012645Hac136c73d80CE0",
  "token": "ETH",
  "amount": "0.5",
  "password": "walletPassword123",
  "note": "转账备注"
}
```

**响应示例**
```json
{
  "code": 200,
  "message": "转账请求已提交",
  "data": {
    "transactionId": "tx_1001",
    "txHash": "0x1234567890abcdef...",
    "status": "pending",
    "estimatedConfirmationTime": 180
  }
}
```

### 获取交易记录

**接口地址**
```
GET /api/v1/transactions
```

**查询参数**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| walletId | string | 否 | 钱包ID筛选 |
| type | string | 否 | 交易类型：send/receive/all |
| status | string | 否 | 交易状态筛选 |
| start_date | string | 否 | 开始日期 YYYY-MM-DD |
| end_date | string | 否 | 结束日期 YYYY-MM-DD |
| page | int | 否 | 页码 |
| size | int | 否 | 每页数量 |

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "transactions": [
      {
        "transactionId": "tx_1001",
        "txHash": "0x1234567890abcdef...",
        "type": "send",
        "fromAddress": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
        "toAddress": "0x8ba1f109551bD432803012645Hac136c73d80CE0",
        "token": "ETH",
        "amount": "0.5",
        "fee": "0.0021",
        "status": "confirmed",
        "confirmations": 12,
        "note": "转账备注",
        "createdAt": "2024-01-15T10:30:00Z",
        "confirmedAt": "2024-01-15T10:33:00Z"
      }
    ],
    "total": 25,
    "page": 1,
    "size": 10
  }
}
```

### 获取交易详情

**接口地址**
```
GET /api/v1/transactions/{transactionId}
```

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "transactionId": "tx_1001",
    "txHash": "0x1234567890abcdef...",
    "blockNumber": 18750000,
    "blockHash": "0xabcdef1234567890...",
    "type": "send",
    "fromAddress": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
    "toAddress": "0x8ba1f109551bD432803012645Hac136c73d80CE0",
    "token": "ETH",
    "amount": "0.5",
    "fee": "0.0021",
    "gasUsed": "21000",
    "gasPrice": "100000000000",
    "status": "confirmed",
    "confirmations": 12,
    "note": "转账备注",
    "createdAt": "2024-01-15T10:30:00Z",
    "confirmedAt": "2024-01-15T10:33:00Z"
  }
}
```

## 支付功能 API

### 创建支付订单

**接口地址**
```
POST /api/v1/payments/orders
```

**请求参数**
```json
{
  "merchantId": "merchant_123",
  "amount": "99.99",
  "currency": "USDT",
  "orderId": "order_20240115001",
  "description": "商品购买",
  "callbackUrl": "https://merchant.com/callback",
  "expireTime": 1800
}
```

**响应示例**
```json
{
  "code": 200,
  "message": "支付订单创建成功",
  "data": {
    "paymentId": "pay_1001",
    "paymentUrl": "https://wallet-project.com/pay/pay_1001",
    "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANS...",
    "amount": "99.99",
    "currency": "USDT",
    "payAddress": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
    "expireAt": "2024-01-15T11:00:00Z"
  }
}
```

### 查询支付状态

**接口地址**
```
GET /api/v1/payments/orders/{paymentId}
```

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "paymentId": "pay_1001",
    "status": "paid",
    "amount": "99.99",
    "currency": "USDT",
    "txHash": "0x1234567890abcdef...",
    "paidAt": "2024-01-15T10:45:00Z"
  }
}
```

### 支付状态说明

| 状态 | 说明 |
|------|------|
| pending | 等待支付 |
| paid | 支付成功 |
| expired | 订单过期 |
| failed | 支付失败 |
| refunded | 已退款 |

## 商户管理 API

### 商户注册

**接口地址**
```
POST /api/v1/merchants/register
```

**请求参数**
```json
{
  "companyName": "示例商户有限公司",
  "contactName": "张三",
  "contact_email": "zhang@example.com",
  "contact_phone": "+86-13812345678",
  "businessType": "电商",
  "website": "https://example.com"
}
```

### 获取商户信息

**接口地址**
```
GET /api/v1/merchants/profile
```

**响应示例**
```json
{
  "code": 200,
  "message": "Success",
  "data": {
    "merchantId": "merchant_123",
    "companyName": "示例商户有限公司",
    "status": "active",
    "apiKey": "mkLive_1234567890abcdef",
    "webhookUrl": "https://example.com/webhook",
    "supportedCurrencies": ["USDT", "ETH", "BTC"],
    "dailyLimit": "100000.00",
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

## WebSocket 实时推送

### 连接地址
```
wss://api.wallet-project.com/ws
```

### 认证
连接时需要传递JWT Token：
```javascript
const ws = new WebSocket('wss://api.wallet-project.com/ws?token=your_jwt_token');
```

### 订阅消息

**订阅钱包余额变化**
```json
{
  "action": "subscribe",
  "channel": "walletBalance",
  "walletId": "w_1001"
}
```

**订阅交易状态更新**
```json
{
  "action": "subscribe",
  "channel": "transaction_status",
  "transactionId": "tx_1001"
}
```

### 推送消息格式

**余额变化通知**
```json
{
  "channel": "walletBalance",
  "data": {
    "walletId": "w_1001",
    "token": "ETH",
    "oldBalance": "1.234567890123456789",
    "newBalance": "1.734567890123456789",
    "change": "+0.5",
    "timestamp": 1693123456789
  }
}
```

**交易状态更新**
```json
{
  "channel": "transaction_status",
  "data": {
    "transactionId": "tx_1001",
    "status": "confirmed",
    "confirmations": 12,
    "timestamp": 1693123456789
  }
}
```

## 错误码说明

### 用户相关错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 1001 | 用户不存在 | 检查用户ID |
| 1002 | 密码错误 | 重新输入正确密码 |
| 1003 | 账户已被锁定 | 联系客服解锁 |
| 1004 | 邮箱已存在 | 使用其他邮箱注册 |

### 钱包相关错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 2001 | 钱包不存在 | 检查钱包ID |
| 2002 | 钱包密码错误 | 重新输入正确密码 |
| 2003 | 余额不足 | 检查账户余额 |
| 2004 | 无效的地址格式 | 检查目标地址格式 |

### 交易相关错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 3001 | 交易金额无效 | 检查交易金额 |
| 3002 | Gas费不足 | 增加Gas费设置 |
| 3003 | 网络拥堵 | 稍后重试 |
| 3004 | 交易已存在 | 避免重复提交 |

### 支付相关错误

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| 4001 | 支付订单不存在 | 检查订单ID |
| 4002 | 支付已过期 | 重新创建支付订单 |
| 4003 | 商户未认证 | 完成商户认证 |
| 4004 | 超出支付限额 | 调整支付金额 |

## 限流策略

### API 限流

| 接口类型 | 限制 | 窗口期 |
|----------|------|--------|
| 登录接口 | 5次/分钟 | 滑动窗口 |
| 查询接口 | 100次/分钟 | 滑动窗口 |
| 交易接口 | 10次/分钟 | 滑动窗口 |
| 支付接口 | 50次/分钟 | 滑动窗口 |

### 限流响应

当触发限流时，API会返回 429 状态码：

```json
{
  "code": 429,
  "message": "请求过于频繁，请稍后再试",
  "data": null,
  "retryAfter": 60
}
```

## 测试环境

### 测试环境信息

| 属性 | 值 |
|------|---|
| **Base URL** | `https://api-test.kawaiichainwallet.com` *(规划中)* |
| **测试账户** | test@kawaiichainwallet.com |
| **测试密码** | Test123456 |

### 测试币申请

可通过以下接口申请测试币：

```
POST /api/v1/test/faucet
```

```json
{
  "walletAddress": "0x742d35Cc6577C4B8A7E13A8A3e3a8cA2b3c5d6e7",
  "token": "ETH",
  "amount": "1.0"
}
```

## 免责声明

⚠️ **重要提醒**:

1. **设计文档**: 本文档为 API 设计规范，不代表当前已实现的功能
2. **开发阶段**: 项目处于早期开发阶段，接口可能随时变更
3. **测试用途**: 请勿在生产环境中使用未正式发布的接口
4. **版本管理**: 正式发布前API版本可能会有重大调整

## 联系支持

- **技术支持**: kawaiichainwallet@gmail.com 
- **API文档**: https://docs.kawaiichainwallet.com *(建设中)*
- **开发者社区**: https://community.kawaiichainwallet.com *(规划中)*