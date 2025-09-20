# 数据库运维

## 数据库优化策略

### 1. 索引策略

```sql
-- 复合索引优化查询
CREATE INDEX idx_transactions_user_status_time ON transactions(user_id, status, created_at DESC);
CREATE INDEX idx_payment_orders_merchant_status ON payment_orders(merchant_id, status);
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, is_read) WHERE is_read = FALSE;

-- 部分索引减少索引大小
CREATE INDEX idx_users_active ON users(user_id) WHERE status = 'active';
CREATE INDEX idx_wallets_active_user ON wallets(user_id) WHERE is_active = TRUE;
```

### 2. 分区策略

```sql
-- 交易表按月分区
CREATE TABLE transactions_y2024m01 PARTITION OF transactions
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- 审计日志表按季度分区
CREATE TABLE audit_logs_y2024q1 PARTITION OF audit_logs
FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
```

### 3. 数据压缩

```sql
-- 历史数据压缩
ALTER TABLE transactions_y2023m12 SET (compression = 'pglz');
ALTER TABLE audit_logs_y2023q4 SET (compression = 'pglz');
```

## 安全配置

### 1. 数据加密

```sql
-- 启用透明数据加密扩展
CREATE EXTENSION IF NOT EXISTS pgcrypto;
COMMENT ON EXTENSION pgcrypto IS 'PostgreSQL加密函数扩展，提供对称和非对称加密功能';

-- 加密敏感字段的示例函数（已添加注释）
CREATE OR REPLACE FUNCTION encrypt_sensitive_data(plaintext TEXT, key TEXT)
RETURNS BYTEA AS $
BEGIN
    RETURN pgp_sym_encrypt(plaintext, key);
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION encrypt_sensitive_data(TEXT, TEXT) IS '敏感数据加密函数，使用PGP对称加密';

CREATE OR REPLACE FUNCTION decrypt_sensitive_data(ciphertext BYTEA, key TEXT)
RETURNS TEXT AS $
BEGIN
    RETURN pgp_sym_decrypt(ciphertext, key);
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION decrypt_sensitive_data(BYTEA, TEXT) IS '敏感数据解密函数，解密PGP对称加密的数据';
```

### 2. 行级安全策略 (RLS)

```sql
-- 启用用户数据的行级安全（已添加注释）
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
COMMENT ON TABLE users IS '用户基础信息表，启用行级安全策略';

ALTER TABLE wallets ENABLE ROW LEVEL SECURITY;
COMMENT ON TABLE wallets IS '用户钱包信息表，启用行级安全策略';

ALTER TABLE transactions ENABLE ROW LEVEL SECURITY;
COMMENT ON TABLE transactions IS '交易记录表，启用行级安全策略';

-- 用户只能访问自己的数据（已添加注释）
CREATE POLICY user_data_policy ON users
    FOR ALL TO app_user
    USING (user_id = current_setting('app.current_user_id')::UUID);
COMMENT ON POLICY user_data_policy ON users IS '用户数据访问策略：用户只能访问自己的数据';

CREATE POLICY wallet_data_policy ON wallets
    FOR ALL TO app_user
    USING (user_id = current_setting('app.current_user_id')::UUID);
COMMENT ON POLICY wallet_data_policy ON wallets IS '钱包数据访问策略：用户只能访问自己的钱包';
```

### 3. 数据脱敏

```sql
-- 创建脱敏视图用于开发/测试环境（已添加注释）
CREATE VIEW users_masked AS
SELECT 
    user_id,
    username,
    CONCAT(LEFT(email, 2), '***', RIGHT(email, 10)) as email_masked,
    CASE WHEN phone IS NOT NULL THEN '***-***-' || RIGHT(phone, 4) END as phone_masked,
    status,
    created_at
FROM users;
COMMENT ON VIEW users_masked IS '用户信息脱敏视图，用于开发和测试环境，隐藏敏感的邮箱和手机号信息';
```

## 性能监控

### 1. 统计信息收集

```sql
-- 自动更新统计信息
ALTER TABLE transactions SET (autovacuum_analyze_scale_factor = 0.02);
ALTER TABLE payment_orders SET (autovacuum_analyze_scale_factor = 0.05);
```

### 2. 查询性能监控

```sql
-- 启用查询统计扩展
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 慢查询监控视图
CREATE VIEW slow_queries AS
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements
WHERE mean_time > 1000 -- 超过1秒的查询
ORDER BY mean_time DESC;
```

## 备份与恢复策略

### 1. 备份配置

```bash
# 全量备份 (每日)
pg_dump -h localhost -U postgres -d kawaii_wallet \
    --format=custom --compress=9 \
    --file=backup_$(date +%Y%m%d).dump

# 增量备份 (WAL归档)
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'
```

### 2. 点时间恢复

```bash
# 恢复到指定时间点
pg_restore -h localhost -U postgres -d kawaii_wallet_restore \
    --create --clean \
    backup_20240115.dump

# 应用WAL日志到指定时间
recovery_target_time = '2024-01-15 14:30:00'
```

## 运维监控

### 1. 关键指标监控

```sql
-- 数据库连接数监控
SELECT count(*) as active_connections 
FROM pg_stat_activity 
WHERE state = 'active';

-- 表空间使用情况
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- 长时间运行的查询
SELECT 
    pid,
    now() - pg_stat_activity.query_start AS duration,
    query 
FROM pg_stat_activity 
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';
```

## 数据生命周期管理

### 1. 数据归档策略

```sql
-- 创建归档表结构（带注释）
CREATE TABLE transactions_archive (
    LIKE transactions INCLUDING ALL
);
COMMENT ON TABLE transactions_archive IS '交易记录归档表，存储超过1年的历史交易数据';

-- 批量归档历史交易数据
CREATE OR REPLACE FUNCTION archive_old_transactions(cutoff_date DATE DEFAULT (CURRENT_DATE - INTERVAL '1 year'))
RETURNS INTEGER AS $
DECLARE
    archived_count INTEGER;
BEGIN
    -- 将超过指定日期的交易记录移动到归档表
    WITH archived_data AS (
        DELETE FROM transactions 
        WHERE created_at < cutoff_date
        RETURNING *
    )
    INSERT INTO transactions_archive 
    SELECT * FROM archived_data;
    
    GET DIAGNOSTICS archived_count = ROW_COUNT;
    
    -- 记录归档操作
    INSERT INTO system_configs (config_key, config_value, config_group, description)
    VALUES (
        'last_transaction_archive',
        archived_count::TEXT,
        'maintenance',
        format('最后一次交易归档：%s，归档记录数：%s', cutoff_date, archived_count)
    )
    ON CONFLICT (config_key) DO UPDATE SET
        config_value = EXCLUDED.config_value,
        description = EXCLUDED.description,
        updated_at = CURRENT_TIMESTAMP;
    
    RETURN archived_count;
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION archive_old_transactions(DATE) IS '交易数据归档函数，将指定日期之前的交易数据移动到归档表';

-- 创建审计日志归档表
CREATE TABLE audit_logs_archive (
    LIKE audit_logs INCLUDING ALL
);
COMMENT ON TABLE audit_logs_archive IS '审计日志归档表，存储超过2年的历史审计记录';

-- 审计日志归档函数
CREATE OR REPLACE FUNCTION archive_old_audit_logs(cutoff_date DATE DEFAULT (CURRENT_DATE - INTERVAL '2 years'))
RETURNS INTEGER AS $
DECLARE
    archived_count INTEGER;
BEGIN
    -- 将超过2年的审计日志移动到归档表
    WITH archived_data AS (
        DELETE FROM audit_logs 
        WHERE created_at < cutoff_date
        RETURNING *
    )
    INSERT INTO audit_logs_archive 
    SELECT * FROM archived_data;
    
    GET DIAGNOSTICS archived_count = ROW_COUNT;
    RETURN archived_count;
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION archive_old_audit_logs(DATE) IS '审计日志归档函数，将指定日期之前的审计日志移动到归档表';
```

### 2. 数据清理任务

```sql
-- 通知数据清理函数
CREATE OR REPLACE FUNCTION cleanup_old_notifications(cutoff_date DATE DEFAULT (CURRENT_DATE - INTERVAL '90 days'))
RETURNS INTEGER AS $
DECLARE
    deleted_count INTEGER;
BEGIN
    -- 删除超过90天的已读通知
    DELETE FROM notifications 
    WHERE created_at < cutoff_date 
    AND is_read = TRUE;
    
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    
    -- 记录清理操作
    INSERT INTO audit_logs (
        action, 
        resource_type, 
        success, 
        metadata
    ) VALUES (
        'CLEANUP',
        'notifications',
        TRUE,
        jsonb_build_object(
            'cutoff_date', cutoff_date,
            'deleted_count', deleted_count,
            'cleanup_type', 'old_read_notifications'
        )
    );
    
    RETURN deleted_count;
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION cleanup_old_notifications(DATE) IS '通知清理函数，删除指定日期之前的已读通知';

-- 过期支付订单清理函数
CREATE OR REPLACE FUNCTION cleanup_expired_payment_orders()
RETURNS INTEGER AS $
DECLARE
    deleted_count INTEGER;
BEGIN
    -- 删除过期超过30天的支付订单
    DELETE FROM payment_orders 
    WHERE status = 'expired' 
    AND expires_at < (CURRENT_TIMESTAMP - INTERVAL '30 days');
    
    GET DIAGNOSTICS deleted_count = ROW_COUNT;
    RETURN deleted_count;
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION cleanup_expired_payment_orders() IS '过期支付订单清理函数，删除过期超过30天的支付订单';

-- 临时数据清理函数
CREATE OR REPLACE FUNCTION cleanup_temp_data()
RETURNS TABLE(
    operation TEXT,
    deleted_count INTEGER
) AS $
BEGIN
    -- 清理过期的临时通知
    RETURN QUERY
    SELECT 
        'expired_notifications'::TEXT,
        cleanup_old_notifications()::INTEGER;
    
    -- 清理过期的支付订单
    RETURN QUERY
    SELECT 
        'expired_payment_orders'::TEXT,
        cleanup_expired_payment_orders()::INTEGER;
    
    -- 清理超过7天的失败交易记录
    DELETE FROM transactions 
    WHERE status = 'failed' 
    AND created_at < (CURRENT_TIMESTAMP - INTERVAL '7 days');
    
    RETURN QUERY
    SELECT 
        'failed_transactions'::TEXT,
        (SELECT changes()::INTEGER);
END;
$ LANGUAGE plpgsql;
COMMENT ON FUNCTION cleanup_temp_data() IS '临时数据清理主函数，执行多种类型的数据清理操作';
```

### 3. 分布式任务调度策略

数据库维护任务将通过分布式任务调度工具（如XXL-JOB、Quartz、Spring Cloud Task等）进行管理：

#### 3.1 推荐的维护任务调度配置

```yaml
# 示例：Spring Boot + Quartz 配置
maintenance-jobs:
  transaction-archive:
    cron: "0 0 2 1 * ?" # 每月1日凌晨2点
    function: "transactionArchiveTask.execute()"
    description: "交易数据归档任务"

  audit-log-archive:
    cron: "0 0 3 1 * ?" # 每月1日凌晨3点
    function: "auditLogArchiveTask.execute()"
    description: "审计日志归档任务"

  notification-cleanup:
    cron: "0 0 1 * * ?" # 每日凌晨1点
    function: "notificationCleanupTask.execute()"
    description: "通知数据清理任务"

  temp-data-cleanup:
    cron: "0 0 4 ? * SUN" # 每周日凌晨4点
    function: "tempDataCleanupTask.execute()"
    description: "临时数据清理任务"

  jwt-blacklist-cleanup:
    cron: "0 0 * * * ?" # 每小时执行
    function: "jwtBlacklistCleanupTask.execute()"
    description: "JWT黑名单清理任务"
```

#### 3.2 Spring Boot实现示例

```java
@Component
@Slf4j
public class DatabaseMaintenanceTask {

    @Autowired
    private MaintenanceService maintenanceService;

    @Scheduled(cron = "0 0 2 1 * ?") // 每月1日凌晨2点
    public void executeTransactionArchive() {
        log.info("开始执行交易数据归档任务");
        try {
            int archived = maintenanceService.archiveOldTransactions();
            log.info("交易数据归档完成，归档记录数: {}", archived);
        } catch (Exception e) {
            log.error("交易数据归档失败", e);
        }
    }

    @Scheduled(cron = "0 0 1 * * ?") // 每日凌晨1点
    public void executeNotificationCleanup() {
        log.info("开始执行通知清理任务");
        try {
            int cleaned = maintenanceService.cleanupOldNotifications();
            log.info("通知清理完成，清理记录数: {}", cleaned);
        } catch (Exception e) {
            log.error("通知清理失败", e);
        }
    }
}
```

#### 3.3 任务监控和管理

- **任务执行日志**: 记录在应用日志和audit_logs表中
- **任务状态监控**: 通过应用监控系统（如Micrometer + Prometheus）监控
- **任务失败告警**: 集成钉钉、企业微信等告警渠道
- **任务可视化管理**: 通过管理后台界面控制任务启停
