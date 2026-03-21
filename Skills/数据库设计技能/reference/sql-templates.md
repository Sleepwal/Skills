# SQL模板库

## 基础表创建模板

### 标准业务表模板

```sql
CREATE TABLE t_{module}_{entity} (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    {business_fields}
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='{table_comment}';
```

### 用户表模板

```sql
CREATE TABLE t_sys_user (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(255) NOT NULL COMMENT '密码（加密）',
    real_name VARCHAR(100) DEFAULT NULL COMMENT '真实姓名',
    email VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
    phone VARCHAR(20) DEFAULT NULL COMMENT '手机号',
    avatar_url VARCHAR(500) DEFAULT NULL COMMENT '头像URL',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-正常',
    last_login_at DATETIME DEFAULT NULL COMMENT '最后登录时间',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_username (username),
    UNIQUE KEY uk_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

### 订单表模板

```sql
CREATE TABLE t_biz_order (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '订单ID',
    order_no VARCHAR(32) NOT NULL COMMENT '订单号',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    total_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '订单总金额',
    discount_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '优惠金额',
    pay_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '实付金额',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '订单状态：1-待支付，2-已支付，3-已发货，4-已完成，5-已取消',
    payment_time DATETIME DEFAULT NULL COMMENT '支付时间',
    ship_time DATETIME DEFAULT NULL COMMENT '发货时间',
    receive_time DATETIME DEFAULT NULL COMMENT '收货时间',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_order_no (order_no),
    KEY idx_user_id (user_id),
    KEY idx_status (status),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

## 索引创建模板

### 单字段索引

```sql
CREATE INDEX idx_{table}_{field} ON t_{table} ({field});
```

### 复合索引

```sql
CREATE INDEX idx_{table}_{field1}_{field2} ON t_{table} ({field1}, {field2});
```

### 唯一索引

```sql
CREATE UNIQUE INDEX uk_{table}_{field} ON t_{table} ({field});
```

## 常用查询模板

### 分页查询

```sql
SELECT * FROM t_{table}
WHERE status = 1
ORDER BY created_at DESC
LIMIT {page_size} OFFSET {offset};

-- ID分页优化版本
SELECT * FROM t_{table}
WHERE id > {last_id}
AND status = 1
ORDER BY id ASC
LIMIT {page_size};
```

### 关联查询

```sql
SELECT a.id, a.name, b.count
FROM t_user a
LEFT JOIN (
    SELECT user_id, COUNT(*) as count
    FROM t_order
    GROUP BY user_id
) b ON a.id = b.user_id
WHERE a.status = 1;
```

### 统计查询

```sql
SELECT
    DATE(created_at) as date,
    COUNT(*) as total,
    SUM(amount) as total_amount
FROM t_order
WHERE created_at >= '{start_date}'
AND created_at < '{end_date}'
GROUP BY DATE(created_at)
ORDER BY date DESC;
```

### 存在性检查

```sql
-- 使用 EXISTS
SELECT *
FROM t_user a
WHERE EXISTS (
    SELECT 1 FROM t_order b
    WHERE b.user_id = a.id AND b.status = 2
);

-- 使用 IN
SELECT *
FROM t_user
WHERE id IN (
    SELECT user_id FROM t_order WHERE status = 2
);
```

## 数据操作模板

### 批量插入

```sql
INSERT INTO t_{table} (field1, field2, field3)
VALUES
    ('value1', 'value2', 'value3'),
    ('value4', 'value5', 'value6'),
    ('value7', 'value8', 'value9');
```

### 批量更新

```sql
UPDATE t_{table}
SET status = 2, updated_at = NOW()
WHERE id IN (1, 2, 3, 4, 5);
```

### 条件更新（防止超扣）

```sql
UPDATE t_user
SET balance = balance - {amount}
WHERE id = {user_id}
AND balance >= {amount};
```

### 软删除

```sql
UPDATE t_{table}
SET deleted_at = NOW()
WHERE id = {id}
AND deleted_at IS NULL;
```

### 数据迁移

```sql
-- 1. 创建新表
CREATE TABLE t_{table}_new LIKE t_{table};

-- 2. 迁移数据
INSERT INTO t_{table}_new SELECT * FROM t_{table};

-- 3. 验证
SELECT COUNT(*) FROM t_{table};
SELECT COUNT(*) FROM t_{table}_new;

-- 4. 切换
RENAME TABLE t_{table} TO t_{table}_old, t_{table}_new TO t_{table};

-- 5. 确认无误后删除
DROP TABLE t_{table}_old;
```

## 表结构变更模板

### 添加字段

```sql
ALTER TABLE t_{table}
ADD COLUMN {field} {type} DEFAULT {default} COMMENT '{comment}',
ALGORITHM=INPLACE, LOCK=NONE;
```

### 添加索引

```sql
CREATE INDEX idx_{table}_{field} ON t_{table} ({field}), ALGORITHM=INPLACE, LOCK=NONE;
```

### 修改字段

```sql
ALTER TABLE t_{table}
MODIFY COLUMN {field} {new_type} DEFAULT {default} COMMENT '{comment}',
ALGORITHM=INPLACE, LOCK=NONE;
```

### 删除字段

```sql
ALTER TABLE t_{table}
DROP COLUMN {field},
ALGORITHM=INPLACE, LOCK=NONE;
```

## 常见场景SQL

### 近7天数据

```sql
SELECT * FROM t_{table}
WHERE created_at >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)
ORDER BY created_at DESC;
```

### 环比增长计算

```sql
SELECT
    DATE(created_at) as date,
    COUNT(*) as today_count,
    LAG(COUNT(*), 1) OVER (ORDER BY DATE(created_at)) as yesterday_count,
    COUNT(*) - LAG(COUNT(*), 1) OVER (ORDER BY DATE(created_at)) as diff
FROM t_{table}
GROUP BY DATE(created_at)
ORDER BY date DESC
LIMIT 30;
```

### 排名查询

```sql
SELECT
    user_id,
    SUM(amount) as total_amount,
    RANK() OVER (ORDER BY SUM(amount) DESC) as ranking
FROM t_order
WHERE status = 2
GROUP BY user_id
LIMIT 100;
```

### 连续天数计算

```sql
SELECT
    user_id,
    COUNT(DISTINCT DATE(login_time)) as login_days
FROM t_user_login
WHERE login_time >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY user_id
HAVING login_days >= 25;
```
