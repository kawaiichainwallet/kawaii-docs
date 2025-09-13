# 技术架构

> 📋 **架构说明**: 本文档描述了 KawaiiChain Wallet 的技术架构设计方案。由于项目处于早期开发阶段，实际实现可能与设计方案有所调整。

## 系统概览

本项目采用现代化的微服务架构，确保系统的可扩展性、可维护性和高可用性。

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Flutter App   │    │   Web Client    │    │  Admin Panel    │
│   (iOS/Android) │    │   (React/Vue)   │    │   (Next.js)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │  API Gateway    │
                    │   (Spring)      │
                    └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  User Service   │    │ Wallet Service  │    │Payment Service  │
│  (Spring Boot)  │    │ (Spring Boot)   │    │ (Spring Boot)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌───────────────────┐
                    │    Database       │
                    │ (PostgreSQL/Redis)│
                    └───────────────────┘
```

## 技术栈详情

### 移动端 - Flutter

**选型原因**
- 跨平台开发，减少维护成本
- 原生性能，流畅的用户体验
- 丰富的加密库支持

**核心技术**
- **Framework**: Flutter 3.8
- **状态管理**: Provider / Riverpod
- **网络请求**: Dio
- **本地存储**: Hive / SQLite
- **安全存储**: Flutter Secure Storage
- **加密库**: crypto、pointycastle

**项目结构**
```
lib/
├── core/           # 核心功能
│   ├── crypto/     # 加密相关
│   ├── network/    # 网络层
│   └── storage/    # 存储层
├── features/       # 功能模块
│   ├── auth/       # 认证
│   ├── wallet/     # 钱包
│   └── payment/    # 支付
├── shared/         # 共享组件
└── main.dart       # 入口文件
```

### 后端服务 - Spring Cloud

**架构选型**
- **注册中心**: Spring Cloud Alibaba Nacos
- **配置中心**: Spring Cloud Alibaba Nacos
- **API网关**: Spring Cloud Gateway
- **熔断器**: Spring Cloud Alibaba Sentinel
- **负载均衡**: Spring Cloud LoadBalancer

**微服务划分**

#### 1. 用户服务 (User Service)
```java
// 主要职责
- 用户注册登录
- 身份验证和授权
- 用户信息管理
- KYC 验证

// 技术栈
- Spring Boot 3.5.x
- Spring Security + JWT
- PostgreSQL 17
- Redis 7.0
```

#### 2. 钱包服务 (Wallet Service)
```java
// 主要职责
- 钱包创建和导入
- 私钥安全管理
- 地址生成和验证
- 交易签名

// 技术栈
- Spring Boot 3.5.x
- Web3j (以太坊)
- BitcoinJ (比特币)
- HSM (硬件安全模块)
```

#### 3. 支付服务 (Payment Service)
```java
// 主要职责
- 转账交易处理
- 商户支付集成
- 交易记录管理
- 风险控制

// 技术栈
- Spring Boot 3.5.x
- RabbitMQ (消息队列)
- MongoDB (交易记录)
- Elasticsearch (搜索)
```

### 前端网站 - Next.js

**项目结构**
```
src/
├── components/     # 通用组件
├── pages/          # 页面路由
├── styles/         # 样式文件
├── lib/            # 工具库
├── hooks/          # 自定义Hooks
└── types/          # TypeScript类型
```

**技术选型**
- **Framework**: Next.js 15+
- **样式**: Tailwind CSS 4
- **状态管理**: Zustand
- **UI组件**: Radix UI / Shadcn/ui
- **表单处理**: React Hook Form
- **数据获取**: SWR / TanStack Query

## 数据库设计

### MySQL - 主数据库

[数据库设计](./DATABASE.md) - 基于PostgreSQL的数据库设计
[数据库运维](./DATABASE-OPERATIONS.md) - 基于PostgreSQL的数据运维计划

### Redis - 缓存层

**用途**
- 会话管理
- API限流
- 交易确认状态
- 热点数据缓存

## 安全架构

### 密钥管理

**分层安全**
```
客户端层:
├── 用户密码加密本地私钥
├── 生物识别二次验证
└── 本地安全存储

服务端层:
├── JWT Token 认证
├── API 请求签名验证
└── 敏感操作多重验证

基础设施:
├── HTTPS 全链路加密
├── VPC 网络隔离
└── WAF 应用防火墙
```

**私钥存储方案**
- 客户端使用 AES-256 加密存储
- 服务端不存储用户私钥
- 支持硬件安全模块(HSM)
- 多重签名企业方案

### 交易安全

**风控策略**
```java
// 交易风控配置
@Component
public class RiskControlConfig {
    
    // 单笔限额
    private BigDecimal singleLimit = new BigDecimal("10000");
    
    // 日累计限额
    private BigDecimal dailyLimit = new BigDecimal("50000");
    
    // 可疑地址黑名单检查
    public boolean checkBlacklist(String address) {
        // 检查地址是否在黑名单中
        return blacklistService.contains(address);
    }
}
```

## 区块链集成

### 多链支持架构

```java
// 区块链适配器接口
public interface BlockchainAdapter {
    String generateAddress();
    BigDecimal getBalance(String address);
    String sendTransaction(TransactionRequest request);
    TransactionStatus getTransactionStatus(String txHash);
}

// 以太坊实现
@Service
public class EthereumAdapter implements BlockchainAdapter {
    @Autowired
    private Web3j web3j;
    
    @Override
    public String sendTransaction(TransactionRequest request) {
        // 以太坊交易实现
    }
}
```

### 交易监控

**实时监控**
- WebSocket 推送交易状态
- 区块确认数跟踪
- 异常交易告警

## 部署架构

### 容器化部署

**Docker 配置**
```dockerfile
# Spring Boot 服务
FROM openjdk:21-jre-slim
COPY target/wallet-service.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

**Kubernetes 配置**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wallet-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wallet-service
  template:
    metadata:
      labels:
        app: wallet-service
    spec:
      containers:
      - name: wallet-service
        image: wallet-service:latest
        ports:
        - containerPort: 8080
```

### 监控和日志

**监控栈**
- **Prometheus** - 指标收集
- **Grafana** - 可视化面板
- **ELK Stack** - 日志分析
- **Jaeger** - 链路追踪

## 性能优化

### 缓存策略

**多级缓存**
1. **客户端缓存** - 用户数据本地缓存
2. **Redis 缓存** - 热点数据缓存
3. **数据库缓存** - MySQL 查询缓存

### 数据库优化

**分库分表策略**
- 按用户ID分表
- 按时间分区存储交易记录
- 读写分离提升性能

## 开发规范

### API 设计规范

```java
// RESTful API 示例
@RestController
@RequestMapping("/api/v1/wallets")
public class WalletController {
    
    @GetMapping("/{id}")
    public ApiResponse<WalletDto> getWallet(@PathVariable Long id) {
        // 获取钱包信息
    }
    
    @PostMapping
    public ApiResponse<WalletDto> createWallet(@RequestBody CreateWalletRequest request) {
        // 创建新钱包
    }
}
```

### 错误处理

```java
// 统一错误响应
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
    
    // 成功响应
    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "Success", data);
    }
    
    // 错误响应
    public static <T> ApiResponse<T> error(int code, String message) {
        return new ApiResponse<>(code, message, null);
    }
}
```