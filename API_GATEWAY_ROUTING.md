# API网关路由配置

## 概述

KawaiiChain Wallet 使用API网关作为统一入口，将客户端请求路由到相应的微服务。

## 网关配置

### 端口配置
- **开发环境**: localhost:8080
- **测试环境**: staging-api.kawaiichainwallet.com
- **生产环境**: api.kawaiichainwallet.com

### 路由规则

#### 1. 认证服务路由 (kawaii-auth:8081)

| 路径模式 | 目标服务 | 描述 |
|---------|---------|------|
| `/api/v1/login` | kawaii-auth:8081/api/v1/login | 用户登录 |
| `/api/v1/login/otp` | kawaii-auth:8081/api/v1/login/otp | OTP登录 |
| `/api/v1/send-login-otp` | kawaii-auth:8081/api/v1/send-login-otp | 发送登录OTP |
| `/api/v1/refresh` | kawaii-auth:8081/api/v1/refresh | 刷新Token |
| `/api/v1/logout` | kawaii-auth:8081/api/v1/logout | 用户登出 |
| `/api/v1/validate` | kawaii-auth:8081/api/v1/validate | 验证Token |
| `/api/v1/health` | kawaii-auth:8081/api/v1/health | 认证服务健康检查 |

#### 2. 用户服务路由 (kawaii-user:8082)

| 路径模式 | 目标服务 | 描述 |
|---------|---------|------|
| `/api/v1/users/**` | kawaii-user:8082/api/v1/users/** | 用户信息管理 |
| `/api/v1/users/profile` | kawaii-user:8082/api/v1/users/profile | 获取/更新用户信息 |
| `/api/v1/users/{userId}` | kawaii-user:8082/api/v1/users/{userId} | 根据ID获取用户 |
| `/api/v1/users/username/{username}` | kawaii-user:8082/api/v1/users/username/{username} | 根据用户名查询 |
| `/api/v1/users/email/{email}` | kawaii-user:8082/api/v1/users/email/{email} | 根据邮箱查询 |
| `/api/v1/users/phone/{phone}` | kawaii-user:8082/api/v1/users/phone/{phone} | 根据手机号查询 |
| `/api/v1/users/check-*` | kawaii-user:8082/api/v1/users/check-* | 检查重复性接口 |
| `/api/v1/users/health` | kawaii-user:8082/api/v1/users/health | 用户服务健康检查 |

#### 3. 核心钱包服务路由 (kawaii-core:8083)

| 路径模式 | 目标服务 | 描述 |
|---------|---------|------|
| `/api/v1/wallets/**` | kawaii-core:8083/api/v1/wallets/** | 钱包管理 |
| `/api/v1/assets/**` | kawaii-core:8083/api/v1/assets/** | 资产管理 |
| `/api/v1/transactions/**` | kawaii-core:8083/api/v1/transactions/** | 交易管理 |

#### 4. 支付服务路由 (kawaii-payment:8084)

| 路径模式 | 目标服务 | 描述 |
|---------|---------|------|
| `/api/v1/payments/**` | kawaii-payment:8084/api/v1/payments/** | 支付管理 |
| `/api/v1/merchants/**` | kawaii-payment:8084/api/v1/merchants/** | 商户管理 |

## 负载均衡策略

### 开发环境
- 单实例部署，无负载均衡

### 生产环境
- Round Robin 轮询算法
- 健康检查间隔：30秒
- 故障转移：自动切换到健康实例

## 安全配置

### CORS设置
```yaml
cors:
  allowed-origins:
    - "http://localhost:3000"    # 开发环境前端
    - "https://kawaiichainwallet.com"  # 生产环境前端
  allowed-methods:
    - GET
    - POST
    - PUT
    - DELETE
    - OPTIONS
  allowed-headers:
    - Content-Type
    - Authorization
    - X-Requested-With
```

### 限流配置
```yaml
rate-limiting:
  default:
    requests-per-minute: 100
  auth:
    requests-per-minute: 20   # 认证接口限流更严格
  payment:
    requests-per-minute: 30   # 支付接口限流
```

## 监控指标

### 关键监控指标
- 请求总数和成功率
- 平均响应时间
- 错误率和错误类型分布
- 各服务的健康状态
- 网关吞吐量

### 告警规则
- 错误率 > 5% 触发告警
- 平均响应时间 > 2秒 触发告警
- 服务不可用触发紧急告警

## Spring Cloud Gateway 配置示例

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 认证服务路由
        - id: auth-service
          uri: http://kawaii-auth:8081
          predicates:
            - Path=/api/v1/login,/api/v1/login/otp,/api/v1/send-login-otp,/api/v1/refresh,/api/v1/logout,/api/v1/validate
          filters:
            - StripPrefix=0

        # 用户服务路由
        - id: user-service
          uri: http://kawaii-user:8082
          predicates:
            - Path=/api/v1/users/**
          filters:
            - StripPrefix=0

        # 核心服务路由
        - id: core-service
          uri: http://kawaii-core:8083
          predicates:
            - Path=/api/v1/wallets/**,/api/v1/assets/**,/api/v1/transactions/**
          filters:
            - StripPrefix=0

        # 支付服务路由
        - id: payment-service
          uri: http://kawaii-payment:8084
          predicates:
            - Path=/api/v1/payments/**,/api/v1/merchants/**
          filters:
            - StripPrefix=0
```

## 客户端使用说明

### 移动端 (Flutter)
```dart
// 所有API调用都通过网关统一入口
const String baseUrl = 'http://localhost:8080';  // 开发环境

// 认证API
POST /api/v1/login
POST /api/v1/login/otp
POST /api/v1/refresh

// 用户API
GET /api/v1/users/profile
PUT /api/v1/users/profile
```

### Web端
```javascript
// 相同的API路径，通过网关路由到对应服务
const apiBase = 'https://api.kawaiichainwallet.com';

// 认证
fetch(`${apiBase}/api/v1/login`, { ... })

// 用户信息
fetch(`${apiBase}/api/v1/users/profile`, { ... })
```

## 故障处理

### 服务降级策略
1. 认证服务故障：返回缓存的Token验证结果
2. 用户服务故障：返回基础用户信息
3. 核心服务故障：禁用钱包操作，显示维护提示
4. 支付服务故障：暂停支付功能

### 熔断器配置
- 故障阈值：50% 错误率
- 熔断时间：30秒
- 半开状态：允许少量请求测试服务恢复

## 部署架构

```
[Client] → [Load Balancer] → [API Gateway] → [Services]
                                   ↓
                           [kawaii-auth:8081]
                           [kawaii-user:8082]
                           [kawaii-core:8083]
                           [kawaii-payment:8084]
```