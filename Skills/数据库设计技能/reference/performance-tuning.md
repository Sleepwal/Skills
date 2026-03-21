# 数据库性能调优指南

## 性能调优原则

1. **先诊断后治疗**：使用EXPLAIN分析慢查询
2. **标本兼治**：解决根本原因而非表面症状
3. **权衡利弊**：在性能、规范、可维护性间平衡
4. **持续监控**：建立性能基线和监控告警

## 查询分析

### EXPLAIN使用

```sql
EXPLAIN SELECT * FROM t_order WHERE user_id = 1;

EXPLAIN结果解读：
- type: 查询类型（const > eq_ref > ref > range > index > ALL）
- possible_keys: 可用的索引
- key: 实际使用的索引
- key_len: 索引长度
- rows: 扫描行数估算
- extra: 额外信息（Using filesort、Using temporary需关注）
```

### 慢查询日志

```sql
-- 查看慢查询配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
```

### 诊断流程

```
1. 识别慢查询
   └── 慢查询日志 / APM工具 / 数据库监控

2. 分析执行计划
   └── EXPLAIN / EXPLAIN ANALYZE

3. 定位问题
   └── 全表扫描 / 索引失效 / 临时表 / 文件排序

4. 制定优化方案
   └── 创建索引 / 重写SQL / 调整结构 / 架构优化

5. 验证效果
   └── 前后对比 / 监控指标
```

## 索引优化

### 索引创建策略

```sql
-- 等值查询字段优先
WHERE status = 1  →  在status建索引

-- 范围查询放最后
WHERE a = 1 AND b > 2 AND c = 3  →  索引(a, b, c) 但c无法有效使用

-- 复合索引顺序
-- 原则：高选择性 > 等值查询 > 范围查询
INDEX idx_user_status (user_id, status)

-- 示例
-- 查询：WHERE user_id = 1 AND status = 1
-- 索引：idx_user_status (user_id, status)  ✓
-- 索引：idx_status_user (status, user_id)  ✗
```

### 索引失效场景

```sql
-- 1. LIKE以通配符开头
SELECT * FROM t_user WHERE name LIKE '%三'  -- 索引失效

-- 2. 对索引字段进行函数运算
SELECT * FROM t_order WHERE YEAR(created_at) = 2024  -- 索引失效
SELECT * FROM t_order WHERE created_at + 1 = NOW()  -- 索引失效

-- 3. 类型转换
SELECT * FROM t_user WHERE id = '1'  -- id是BIGINT，字符串'1'会转换

-- 4. OR条件
SELECT * FROM t_user WHERE id = 1 OR name = '张三'  -- 部分失效
-- 改写为 UNION
SELECT * FROM t_user WHERE id = 1
UNION ALL
SELECT * FROM t_user WHERE name = '张三' AND id IS NULL

-- 5. NOT条件
SELECT * FROM t_user WHERE status != 1  -- 索引失效
SELECT * FROM t_user WHERE status NOT IN (1, 2)  -- 索引失效

-- 6. IS NULL/IS NOT NULL
SELECT * FROM t_user WHERE deleted_at IS NULL  -- 可能不使用索引
```

### 索引维护

```sql
-- 分析索引使用情况
SELECT
    object_schema,
    object_name,
    index_name,
    count_star,
    count_read,
    count_write
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema = 'your_database'
AND object_name = 'your_table';

-- 删除未使用索引
ALTER TABLE t_order DROP INDEX idx_old_index;

-- 查看索引碎片
SHOW TABLE STATUS FROM database LIKE 't_order';

-- 定期OPTIMIZE重建索引
OPTIMIZE TABLE t_order;
```

## SQL优化

### SELECT优化

```sql
-- ✗ 低效写法
SELECT * FROM t_order WHERE order_no = 'xxx';  -- SELECT *

-- ✓ 高效写法
SELECT id, order_no, amount, status FROM t_order WHERE order_no = 'xxx';

-- ✗ 低效写法
SELECT * FROM t_order ORDER BY id DESC LIMIT 1000000, 10;  -- 深度分页

-- ✓ 高效写法 - ID分页
SELECT * FROM t_order WHERE id > 1000000 ORDER BY id LIMIT 10;

-- ✓ 高效写法 - 延迟关联
SELECT a.* FROM t_order a
INNER JOIN (SELECT id FROM t_order ORDER BY id LIMIT 1000000, 10) b
ON a.id = b.id;
```

### JOIN优化

```sql
-- ✗ 低效写法
SELECT a.*, b.*, c.*
FROM t_order a
JOIN t_user b ON a.user_id = b.id
JOIN t_product c ON a.product_id = c.id
WHERE b.status = 1;

-- ✓ 高效写法 - 先筛选后关联
SELECT a.*, b.name, c.title
FROM t_order a
INNER JOIN t_user b ON a.user_id = b.id AND b.status = 1
INNER JOIN t_product c ON a.product_id = c.id
WHERE a.status = 2;

-- ✓ 高效写法 - 应用层处理
-- 在应用层分别查询，减少JOIN复杂度
```

### UPDATE/DELETE优化

```sql
-- ✗ 低效写法 - 逐条更新
UPDATE t_user SET status = 2 WHERE id = 1;
UPDATE t_user SET status = 2 WHERE id = 2;
UPDATE t_user SET status = 2 WHERE id = 3;

-- ✓ 高效写法 - 批量更新
UPDATE t_user SET status = 2 WHERE id IN (1, 2, 3);

-- ✗ 低效写法 - 循环删除
for (id in ids) {
    DELETE FROM t_order WHERE id = id;
}

-- ✓ 高效写法 - 批量删除
DELETE FROM t_order WHERE id IN (1, 2, 3, 4, 5);

-- ✓ 安全写法 - 带锁检查
UPDATE t_order
SET status = 2, updated_at = NOW()
WHERE id = 1 AND status = 1;  -- 检查原状态

-- ✓ 金额操作 - 防超扣
UPDATE t_user
SET balance = balance - 100
WHERE id = 1 AND balance >= 100;  -- 检查余额
```

### 批量操作

```sql
-- ✗ 低效写法 - 循环插入
for (order in orders) {
    INSERT INTO t_order (user_id, amount) VALUES (order.userId, order.amount);
}

-- ✓ 高效写法 - 批量插入
INSERT INTO t_order (user_id, amount) VALUES
(1, 100),
(2, 200),
(3, 300);

-- 限制批量大小，建议每次500-1000条
```

## 表结构优化

### 字段类型优化

```sql
-- ✗ 低效
CREATE TABLE t_user (
    phone VARCHAR(100) DEFAULT NULL,
    amount VARCHAR(50) DEFAULT NULL,
    created_date VARCHAR(20) DEFAULT NULL
);

-- ✓ 高效
CREATE TABLE t_user (
    phone VARCHAR(20) DEFAULT NULL,
    amount DECIMAL(12,2) DEFAULT 0.00,
    created_date DATE DEFAULT NULL
);
```

### 大表优化策略

```sql
-- 1. 分区表
CREATE TABLE t_order (
    id BIGINT NOT NULL,
    order_no VARCHAR(32),
    created_at DATETIME,
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- 2. 历史数据归档
-- 将N个月前的数据迁移到历史表
INSERT INTO t_order_history SELECT * FROM t_order
WHERE created_at < DATE_SUB(NOW(), INTERVAL 12 MONTH);

DELETE FROM t_order WHERE created_at < DATE_SUB(NOW(), INTERVAL 12 MONTH);
```

### 读写分离

```sql
-- 读从库
SELECT * FROM t_order WHERE id = 1;  -- 读从库

-- 写主库
INSERT INTO t_order (...) VALUES (...);  -- 写主库
UPDATE t_order SET ... WHERE id = 1;  -- 写主库

-- 注意：读写分离后同一事务内不要混用
```

## 连接优化

```sql
-- 查看连接数
SHOW STATUS LIKE 'Threads_connected';
SHOW VARIABLES LIKE 'max_connections';

-- 设置连接池大小
-- MySQL: max_connections = 100-200
-- 应用端: 连接池大小 = CPU核心数 * 2 + 磁盘数

-- 短连接优化
-- 避免频繁建立和关闭连接
-- 使用连接池管理连接
```

## 缓存策略

### 多级缓存

```
请求 → Redis缓存 → 数据库
         ↓
      缓存未命中
         ↓
      查询数据库 → 写入Redis → 返回
```

### 缓存模式

```sql
-- Cache-Aside（旁路缓存）
-- 读：先读缓存，未命中查数据库并写入缓存
-- 写：先写数据库，再删除缓存

-- Read/Write Through
-- 读写操作都经过缓存层，缓存负责同步

-- Write Behind
-- 写操作只写缓存，异步写数据库
```

### 缓存使用注意

```sql
-- 避免缓存雪崩
-- 不同key设置不同过期时间
SET key EX (300 + RAND() % 100);

-- 避免缓存穿透
-- 对不存在的数据也缓存（设置短过期时间）
-- 使用布隆过滤器

-- 避免缓存击穿
-- 使用互斥锁
-- 设置热点数据永不过期
```

## 监控指标

### 关键指标

```sql
-- QPS
SHOW GLOBAL STATUS LIKE 'Questions';
SHOW GLOBAL STATUS LIKE 'Com_select';

-- TPS
SHOW GLOBAL STATUS LIKE 'Com_insert';
SHOW GLOBAL STATUS LIKE 'Com_update';
SHOW GLOBAL STATUS LIKE 'Com_delete';

-- 连接数
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Max_used_connections';

-- 缓存命中率
SHOW GLOBAL STATUS LIKE 'Qcache_hits';
SHOW GLOBAL STATUS LIKE 'Qcache_inserts';

-- 锁等待
SHOW ENGINE INNODB STATUS;
```

### 性能基线

```
基准指标：
- 简单查询响应时间：< 10ms
- 复杂查询响应时间：< 100ms
- 插入/更新响应时间：< 20ms
- 并发连接数：< 100
- CPU使用率：< 70%
- 磁盘IO使用率：< 70%
```

## 常见问题处理

### CPU使用率高

```sql
-- 1. 查看当前连接
SHOW PROCESSLIST;

-- 2. 查看慢查询
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;

-- 3. 查看锁等待
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 4. 临时解决方案
KILL CONNECTION {connection_id};
```

### 内存使用率高

```sql
-- 1. 查看内存使用
SHOW ENGINE INNODB STATUS;

-- 2. 调整缓冲池大小
SET GLOBAL innodb_buffer_pool_size = 8589934592;  -- 8GB

-- 3. 清理缓存
RESET QUERY CACHE;
```

### 锁等待严重

```sql
-- 1. 查看锁情况
SELECT * FROM information_schema.INNODB_LOCKS;

-- 2. 查看事务
SELECT * FROM information_schema.INNODB_TRX;

-- 3. 解决办法
-- 缩短事务时间
-- 减少隔离级别（如读已提交）
-- 优化索引减少锁范围
```
