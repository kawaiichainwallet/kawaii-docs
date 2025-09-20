# Kawaii Server 功能特性

> 🚀 **后端服务**: 本文档详细描述了 KawaiiChain Wallet Spring Boot 微服务集群的功能特性和技术实现。

## 项目概览

### 技术栈
- **Framework**: Spring Boot 3.5.5
- **Java版本**: Java 21
- **微服务**: Spring Cloud 2025.0.0
- **数据库**: PostgreSQL 17 + Redis 7.0
- **ORM**: MyBatis-Plus 3.5.8
- **API文档**: SpringDoc OpenAPI 2.2.0
- **对象映射**: MapStruct 1.5.5.Final

### 微服务架构
```
kawaii-server/
├── kawaii-common/        # 通用模块
├── kawaii-gateway/       # API网关 (8090)
├── kawaii-user/          # 用户服务 (8082)
├── kawaii-core/          # 钱包核心 (8083)
├── kawaii-payment/       # 支付服务 (8084)
└── pom.xml              # 父项目配置
```

## 已实现功能

### 🌐 API网关 (kawaii-gateway)

#### 统一认证
- ✅ JWT Token验证
- ✅ 访问令牌自动验证
- ✅ 刷新令牌机制
- ✅ 用户上下文提取

#### 路由管理
- ✅ 动态路由配置
- ✅ 路径重写规则
- ✅ 服务负载均衡
- ✅ 路由优先级控制

#### 安全控制
- ✅ 公开路径白名单
- ✅ 受保护路径验证
- ✅ 角色权限检查
- ✅ 管理员路径保护

#### 限流和监控
- ✅ IP限流配置
- ✅ Redis限流存储
- ✅ CORS跨域处理
- ✅ 请求日志记录

#### 服务间通信
- ✅ OpenFeign客户端
- ✅ 内部Token认证
- ✅ 用户上下文传递
- ✅ 请求追踪

### 👤 用户服务 (kawaii-user)

#### 用户认证 (集成认证功能)
- ✅ 用户名/邮箱/手机号登录
- ✅ 手机OTP登录
- ✅ JWT Token生成和验证
- ✅ 刷新Token机制
- ✅ 用户登出

#### 用户管理
- ✅ 用户注册（手机/邮箱）
- ✅ OTP验证码发送
- ✅ 用户信息CRUD
- ✅ 用户资料更新
- ✅ 用户状态管理

#### 用户查询
- ✅ 根据用户名查询
- ✅ 根据邮箱查询
- ✅ 根据手机号查询
- ✅ 用户存在性检查
- ✅ 用户列表查询（管理员）

#### 安全特性
- ✅ 密码哈希加盐
- ✅ JWT签名验证
- ✅ 登录失败锁定
- ✅ 审计日志记录
- ✅ 最后登录追踪

#### 数据模型
- ✅ User实体（用户基础信息）
- ✅ UserProfile实体（用户详细资料）
- ✅ AuditLog实体（审计日志）
- ✅ DTO对象映射（MapStruct）

### 🛠️ 通用模块 (kawaii-common)

#### 响应封装
- ✅ R<T>统一响应格式
- ✅ 成功/失败响应构建
- ✅ 错误代码枚举
- ✅ 追踪ID支持

#### 工具类
- ✅ JWT工具类
- ✅ 请求工具类
- ✅ 验证工具类
- ✅ 加密工具类

#### 异常处理
- ✅ 全局异常处理器
- ✅ 业务异常定义
- ✅ 参数验证异常
- ✅ 安全异常处理

#### 配置管理
- ✅ OpenFeign统一配置
- ✅ 内部认证拦截器
- ✅ 请求追踪
- ✅ 日志配置

### 💰 钱包核心服务 (kawaii-core)

#### 服务架构
- ✅ Spring Boot启动类
- ✅ OpenFeign客户端配置
- ✅ 用户服务调用
- ✅ 内部认证支持

#### 基础功能
- ✅ 服务健康检查
- ✅ API文档集成
- ✅ 错误处理
- ✅ 日志记录

### 💳 支付服务 (kawaii-payment)

#### 服务架构
- ✅ Spring Boot启动类
- ✅ OpenFeign客户端配置
- ✅ 用户服务调用
- ✅ 钱包服务调用

#### 服务间集成
- ✅ 用户信息验证
- ✅ 钱包余额查询
- ✅ 转账执行调用
- ✅ 内部Token认证

## 开发中功能

### 💰 钱包核心功能
- 🚧 钱包创建和导入
- 🚧 私钥安全管理
- 🚧 地址生成和验证
- 🚧 交易签名

### 🔗 区块链集成
- 🚧 以太坊网络支持
- 🚧 比特币网络支持
- 🚧 多链钱包管理
- 🚧 交易广播

### 💳 支付功能
- 🚧 转账交易处理
- 🚧 商户支付集成
- 🚧 交易记录管理
- 🚧 风险控制

### 📊 数据分析
- 🚧 用户行为分析
- 🚧 交易统计
- 🚧 系统监控
- 🚧 性能指标

## 计划功能

### 🏪 商户系统
- 📋 商户注册和审核
- 📋 商户支付接口
- 📋 商户后台管理
- 📋 结算系统

### 💰 DeFi集成
- 📋 去中心化交易所集成
- 📋 流动性挖矿
- 📋 Staking服务
- 📋 收益管理

### 🎯 高级功能
- 📋 多重签名钱包
- 📋 硬件钱包支持
- 📋 冷钱包管理
- 📋 企业级功能

### 🛡️ 安全增强
- 📋 硬件安全模块(HSM)
- 📋 风险控制系统
- 📋 反洗钱(AML)
- 📋 合规管理

## 技术实现细节

### 数据库设计
```sql
-- 用户相关表
users                 # 用户基础信息
user_profiles         # 用户详细资料
audit_logs           # 审计日志

-- 钱包相关表（规划中）
wallets              # 钱包信息
wallet_addresses     # 钱包地址
transactions         # 交易记录

-- 支付相关表（规划中）
payments             # 支付记录
merchant_accounts    # 商户账户
settlement_records   # 结算记录
```

### API设计规范
```yaml
# 统一响应格式
R<T>:
  code: Integer        # 状态码
  msg: String         # 消息
  data: T             # 数据
  timestamp: DateTime # 时间戳
  traceId: String     # 追踪ID
  success: Boolean    # 成功标识

# 路由规则
/api/v1/login                    # 用户登录
/api/v1/users/**                 # 用户管理
/api/v1/wallet/**                # 钱包功能
/api/v1/payment/**               # 支付功能
```

### 服务间通信
```java
// OpenFeign客户端示例
@FeignClient(name = "kawaii-user", url = "${app.services.user.url}")
public interface UserServiceClient {
    @GetMapping("/api/v1/users/{userId}")
    R<UserInfo> getUserInfo(@PathVariable String userId,
                           @RequestHeader("X-Internal-Token") String token);
}
```

### 安全机制
- JWT Token认证
- 密码BCrypt哈希
- 内部服务Token
- 请求追踪和审计
- 角色权限控制

### 配置管理
```yaml
# Gateway配置
app.security.routes:
  public-paths: ["/api/v1/login", "/api/v1/users/register/**"]
  protected-paths: ["/api/v1/users/profile/**"]
  admin-paths: ["/api/v1/admin/**"]

# JWT配置
app.jwt:
  secret: kawaii-chain-wallet-secret-key-2025
  access-token-expiration: 900000    # 15分钟
  refresh-token-expiration: 604800000 # 7天
```

## 部署架构

### 服务端口
- kawaii-gateway: 8090
- kawaii-user: 8082
- kawaii-core: 8083
- kawaii-payment: 8084

### 依赖服务
- PostgreSQL: 5432
- Redis: 6379

### 监控端点
- Health Check: /actuator/health
- Metrics: /actuator/metrics
- API文档: /swagger-ui.html

## 开发规范

### 代码规范
- 使用MapStruct进行对象映射
- MyBatis-Plus简化数据访问
- 统一异常处理
- RESTful API设计

### 数据库规范
- 使用UUID作为主键
- 软删除机制
- 创建和更新时间戳
- 审计日志记录

### 安全规范
- 密码强度要求
- JWT Token管理
- 内部服务认证
- 敏感数据加密

### 测试策略
- 单元测试覆盖
- 集成测试
- API测试
- 性能测试

## 版本信息

- **当前版本**: v0.1.0-dev
- **Spring Boot**: 3.5.5
- **Java**: 21
- **开发状态**: 早期开发阶段

---

*最后更新: 2025-09-20*