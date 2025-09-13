# 贡献指南

> 🚧 **项目状态**: KawaiiChain Wallet 目前处于早期开发阶段，我们热烈欢迎社区开发者的参与和贡献！

欢迎参与 KawaiiChain Wallet 项目的开发！本文档将指导你如何为项目做出贡献。

## ⚠️ 贡献者须知

### 项目性质声明

在参与贡献之前，请务必了解：

- **技术研究项目**: 本项目仅供技术学习、研究和教育目的
- **合规责任**: 贡献者应确保其贡献内容符合所在地区的法律法规
- **开源协议**: 所有贡献将遵循项目的开源许可证
- **风险认知**: 贡献者应了解区块链技术的相关风险和法律考量

### 贡献原则

1. **技术导向**: 专注于技术实现和代码质量
2. **教育目的**: 确保代码和文档具有教育价值
3. **合规意识**: 避免涉及具体的金融服务实现
4. **安全优先**: 重视代码安全和用户隐私保护

## 当前开发重点

由于项目处于早期阶段，我们当前的开发重点是：

- 🔧 **核心架构搭建** - 建立稳定的技术架构
- 🏗️ **基础功能开发** - 实现钱包的核心功能
- 📚 **文档完善** - 完善项目文档和开发指南
- 🧪 **测试框架** - 建立完整的测试体系

## 贡献方式

### 🐛 报告Bug
发现问题时，请创建详细的Issue报告

### 💡 功能建议  
提出新功能想法和改进建议

### 📝 文档改进
完善项目文档、API说明、示例代码

### 💻 代码贡献
修复Bug、实现新功能、性能优化

### 🧪 测试支持
编写测试用例、进行兼容性测试

### 🎨 UI/UX 设计
界面设计、用户体验优化

## 新手友好任务

我们为新加入的贡献者准备了一些入门任务：

- 📖 **文档翻译** - 将文档翻译成其他语言
- 🐛 **简单Bug修复** - 标记为 `good first issue` 的问题
- 📝 **代码注释** - 为现有代码添加详细注释
- 🧪 **单元测试** - 为现有功能编写测试用例
- 🎨 **UI改进** - 界面细节优化

## 开发环境搭建

### 系统要求

| 组件 | 版本要求 |
|------|----------|
| Java | 21+ |
| Node.js | 18+ |
| Flutter | 3.35 |
| Docker | 20.0+ |
| Git | 2.30+ |

### 克隆代码

```bash
# 克隆主仓库
git clone https://github.com/kawaiichainwallet/kawaii-server.git
git clone https://github.com/kawaiichainwallet/kawaii-mobile.git
git clone https://github.com/kawaiichainwallet/kawaii-website.git

# 或者克隆你Fork的仓库
git clone https://github.com/your-username/kawaii-server.git
cd kawaii-server
git remote add upstream https://github.com/kawaiichainwallet/kawaii-server.git
```

### 后端开发环境

```bash
cd kawaii-server

# 启动依赖服务
docker-compose up -d postgres redis

# 安装依赖
./mvnw clean install

# 启动开发服务
./mvnw spring-boot:run -pl kawaii-gateway
```

### 移动端开发环境

```bash
cd kawaii-mobile

# 检查Flutter环境
flutter doctor

# 获取依赖
flutter pub get

# 运行应用
flutter run
```

### 前端开发环境

```bash
cd kawaii-website

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

## 开发规范

### 分支管理

**分支命名规范**
```bash
# 功能开发
feature/用户认证功能
feature/多重签名钱包

# Bug修复  
fix/修复转账金额计算错误
fix/解决登录token过期问题

# 文档更新
docs/更新API文档
docs/添加部署指南

# 重构代码
refactor/优化交易服务架构
refactor/重构用户管理模块
```

**开发流程**
```bash
# 1. 创建功能分支
git checkout -b feature/新功能名称

# 2. 开发完成后提交
git add .
git commit -m "feat: 添加新功能描述"

# 3. 推送到远程分支
git push origin feature/新功能名称

# 4. 创建Pull Request
```

### 提交信息规范

**格式要求**
```
<类型>(<范围>): <描述>

[可选的正文]

[可选的脚注]
```

**类型说明**
- `feat`: 新功能
- `fix`: 修复Bug
- `docs`: 文档更新
- `style`: 代码格式化
- `refactor`: 重构代码
- `test`: 测试相关
- `chore`: 构建工具、依赖更新

**示例**
```bash
feat(wallet): 添加多重签名钱包支持

- 实现2-of-3多重签名方案
- 添加签名验证逻辑
- 更新钱包创建接口

Closes #123
```

### 代码规范

#### Java 代码规范

**命名约定**
```java
// 类名：帕斯卡命名法
public class WalletService {
    
    // 方法名：驼峰命名法
    public Wallet createWallet(CreateWalletRequest request) {
        // 变量名：驼峰命名法
        String walletAddress = generateAddress();
        
        // 常量：全大写下划线分隔
        private static final String DEFAULT_CHAIN = "ethereum";
    }
}
```

**注释规范**
```java
/**
 * 创建新钱包
 * 
 * @param request 钱包创建请求参数
 * @return 创建的钱包信息
 * @throws WalletException 当钱包创建失败时抛出
 */
public Wallet createWallet(CreateWalletRequest request) throws WalletException {
    // 验证请求参数
    validateRequest(request);
    
    // 生成钱包地址
    String address = generateAddress(request.getChainType());
    
    return wallet;
}
```

#### Flutter 代码规范

**命名约定**
```dart
// 类名：帕斯卡命名法
class WalletService {
  
  // 方法名和变量名：驼峰命名法
  Future<Wallet> createWallet(String name) async {
    final walletData = await _generateWallet();
    return Wallet.fromJson(walletData);
  }
  
  // 私有成员：下划线前缀
  final ApiClient _apiClient;
  
  // 常量：驼峰命名法
  static const String defaultChain = 'ethereum';
}
```

**Widget 结构**
```dart
class WalletListPage extends StatefulWidget {
  const WalletListPage({super.key});

  @override
  State<WalletListPage> createState() => _WalletListPageState();
}

class _WalletListPageState extends State<WalletListPage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('我的钱包'),
      ),
      body: const WalletList(),
    );
  }
}
```

#### TypeScript 代码规范

**命名约定**
```typescript
// 接口：帕斯卡命名法，I前缀
interface IWalletService {
  createWallet(params: CreateWalletParams): Promise<Wallet>;
}

// 类型：帕斯卡命名法
type WalletStatus = 'active' | 'inactive' | 'locked';

// 函数和变量：驼峰命名法
const walletService = new WalletService();

// 常量：全大写下划线分隔
const API_BASE_URL = 'https://api.wallet-project.com';
```

**函数定义**
```typescript
/**
 * 创建新钱包
 * @param params 创建参数
 * @returns 钱包信息
 */
async function createWallet(params: CreateWalletParams): Promise<Wallet> {
  try {
    const response = await apiClient.post('/wallets', params);
    return response.data;
  } catch (error) {
    throw new WalletCreationError('钱包创建失败', error);
  }
}
```

## 测试要求

### 单元测试

**Java 测试**
```java
@SpringBootTest
class WalletServiceTest {
    
    @Autowired
    private WalletService walletService;
    
    @Test
    @DisplayName("应该成功创建以太坊钱包")
    void shouldCreateEthereumWallet() {
        // Given
        CreateWalletRequest request = CreateWalletRequest.builder()
            .name("测试钱包")
            .chainType(ChainType.ETHEREUM)
            .password("test123")
            .build();
        
        // When
        Wallet wallet = walletService.createWallet(request);
        
        // Then
        assertThat(wallet).isNotNull();
        assertThat(wallet.getChainType()).isEqualTo(ChainType.ETHEREUM);
        assertThat(wallet.getAddress()).startsWith("0x");
    }
}
```

**Flutter 测试**
```dart
void main() {
  group('WalletService', () {
    late WalletService walletService;
    
    setUp(() {
      walletService = WalletService();
    });
    
    test('应该成功创建钱包', () async {
      // Arrange
      const walletName = '测试钱包';
      
      // Act
      final wallet = await walletService.createWallet(walletName);
      
      // Assert
      expect(wallet, isNotNull);
      expect(wallet.name, equals(walletName));
      expect(wallet.address, startsWith('0x'));
    });
  });
}
```

### 集成测试

**API 测试**
```javascript
describe('钱包API测试', () => {
  test('POST /api/v1/wallets 应该创建新钱包', async () => {
    const response = await request(app)
      .post('/api/v1/wallets')
      .set('Authorization', `Bearer ${authToken}`)
      .send({
        name: '测试钱包',
        chain_type: 'ethereum',
        password: 'test123'
      })
      .expect(200);
    
    expect(response.body.data).toHaveProperty('wallet_id');
    expect(response.body.data.chain_type).toBe('ethereum');
  });
});
```

### 测试覆盖率要求

| 组件 | 最低覆盖率 |
|------|------------|
| 核心业务逻辑 | 90% |
| API接口 | 85% |
| 工具类 | 80% |
| UI组件 | 70% |

## 代码审查

### Pull Request 要求

**PR模板**
```markdown
## 变更描述
简要描述本次变更的内容和目的

## 变更类型
- [ ] 新功能
- [ ] Bug修复
- [ ] 性能优化
- [ ] 文档更新
- [ ] 重构
- [ ] 其他

## 测试
- [ ] 已添加单元测试
- [ ] 已添加集成测试
- [ ] 手动测试通过

## 检查清单
- [ ] 代码符合规范
- [ ] 已更新相关文档
- [ ] 没有合并冲突
- [ ] 所有测试通过
```

### 审查要点

**代码质量**
- 逻辑清晰，命名规范
- 没有重复代码
- 适当的错误处理
- 性能考虑

**安全性**
- 输入验证
- 权限检查
- 敏感信息处理
- SQL注入防护

**可维护性**
- 代码结构合理
- 注释充分
- 测试覆盖充分
- 文档完整

## 发布流程

### 版本命名规范

遵循语义化版本控制 (SemVer)：

```
主版本号.次版本号.修订号

1.0.0 - 初始版本
1.1.0 - 新增功能
1.1.1 - Bug修复
2.0.0 - 重大变更
```

### 发布步骤

**1. 准备发布**
```bash
# 更新版本号
git checkout develop
git pull origin develop

# 创建发布分支
git checkout -b release/v1.1.0
```

**2. 测试验证**
```bash
# 运行完整测试套件
npm run test:full
mvn test
flutter test

# 构建发布版本
npm run build:prod
mvn clean package
flutter build apk --release
```

**3. 创建标签**
```bash
# 合并到主分支
git checkout main
git merge release/v1.1.0

# 创建版本标签
git tag -a v1.1.0 -m "Release version 1.1.0"
git push origin main --tags
```

**4. 部署上线**
```bash
# 自动化部署脚本
./scripts/deploy-production.sh v1.1.0
```

## 社区参与

### 讨论渠道

- **GitHub Discussions** - 功能讨论、技术交流
- **Issue Tracker** - Bug报告、功能请求

### 认可贡献者

我们会通过以下方式认可贡献者：

- **Contributors列表** - README.md中展示
- **Release Notes** - 发布说明中感谢
- **年度贡献奖** - 优秀贡献者奖励
- **技术博客** - 分享贡献者的技术文章

## 常见问题

### Q: 如何选择适合的Issue？

**A**: 初次贡献者建议选择标记为 `good first issue` 的问题，有一定经验后可以尝试 `help wanted` 标签的Issue。

### Q: 代码审查周期多长？

**A**: 通常在2-3个工作日内会有初步反馈，复杂的PR可能需要更长时间。

### Q: 如何处理合并冲突？

**A**: 
```bash
# 更新本地主分支
git checkout main
git pull upstream main

# 切换到功能分支并变基
git checkout feature/your-feature
git rebase main

# 解决冲突后继续
git add .
git rebase --continue
```

### Q: 测试失败怎么办？

**A**: 
1. 检查错误信息，确定失败原因
2. 在本地重现并修复问题
3. 确保所有测试通过后再推送
4. 如需帮助，在PR中提问

## 行为准则

我们致力于为所有参与者创造一个友好、包容的环境：

- **尊重他人** - 保持专业和礼貌
- **建设性反馈** - 提供有助于改进的建议
- **协作精神** - 乐于分享和学习
- **质量第一** - 追求代码和文档的高质量

## 联系方式

如有任何问题或建议，欢迎联系：

- **项目维护者**: kawaiichainwallet@gmail.com
- **GitHub Issues**: [创建Issue](https://github.com/kawaiichainwallet/wallet-docs/issues/new)

感谢你对项目的贡献！🙏