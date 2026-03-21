---
name: "database-design"
description: "数据库设计技能包，提供完整的数据库设计指导。适用于新建数据库、设计规范查询、数据库优化咨询等场景。"
---

# 数据库设计技能

本技能提供全面的数据库设计指导，涵盖从需求分析到物理设计的完整流程，帮助你构建高效、可靠、可扩展的数据库系统。

## 核心概念

### 1. 数据模型基础

**实体（Entity）**
- 现实世界中可以相互区分的事物
- 通过属性（Attribute）来描述
- 例如：用户、订单、产品

**关系（Relationship）**
- 实体之间的联系
- 类型：一对一（1:1）、一对多（1:N）、多对多（M:N）
- 通过外键（FOREIGN KEY）实现

**主键（Primary Key）**
- 唯一标识表中每一行
- 不可为空（NOT NULL）
- 具有唯一性（UNIQUE）
- 推荐使用代理键（Surrogate Key）而非自然键

**外键（Foreign Key）**
- 建立表间关联的字段
- 引用另一个表的主键
- 确保引用完整性

### 2. 规范化理论

**第一范式（1NF）**
- 原子性：每个字段不可再分
- 示例：`地址` 字段应拆分为 `省`、`市`、`区`、`详细地址`

**第二范式（2NF）**
- 满足1NF
- 非主键字段完全依赖于主键（消除部分依赖）
- 示例：订单明细表（订单号+商品号）为主键，价格应完全依赖于主键，而非仅依赖商品号

**第三范式（3NF）**
- 满足2NF
- 非主键字段之间不存在传递依赖
- 示例：学生表中学号为主键，班级名称依赖学号而非直接依赖

**BC范式（BCNF）**
- 满足3NF
- 任何时候只有一个候选键能决定其他属性
- 处理多值依赖问题

**规范化程度选择**
| 场景 | 推荐范式 | 原因 |
|------|---------|------|
| OLTP系统 | 3NF/B CNF | 写操作频繁，需减少更新异常 |
| OLAP系统 | 2NF/1NF | 读操作频繁，允许适当冗余提升查询性能 |
| 简单表结构 | 保持3NF | 平衡性能与维护性 |

### 3. 索引设计

**索引类型**
- B-Tree索引：默认索引类型，适合等值查询和范围查询
- Hash索引：适合等值查询，不支持范围查询
- 全文索引：适合文本搜索
- 复合索引：多字段组合，遵循最左前缀原则

**索引设计原则**
- 为WHERE、JOIN、ORDER BY、GROUP BY涉及的字段建立索引
- 选择性（Selectivity）高的字段优先
- 避免在频繁更新的字段上建索引
- 控制索引数量，单表不超过5-7个

**索引失效场景**
- 使用LIKE以通配符开头（如`%abc`）
- 对索引字段进行函数运算
- 类型转换
- OR条件导致索引失效

## 设计方法论

### 1. 需求分析阶段

**收集信息**
- 业务需求文档
- 现有系统数据字典
- 用户访谈和调研
- 报表和分析需求

**产出物**
- 数据概念模型（ER图）
- 功能需求列表
- 性能要求指标

### 2. 概念设计阶段

**任务**
- 识别实体和关系
- 确定实体属性
- 绘制ER图
- 验证业务场景覆盖

**ER图要素**
```
┌─────────┐         ┌─────────┐
│  实体A  │ 1 ── N │  实体B  │
└─────────┘         └─────────┘
     │
     │ N
     ▼
┌─────────┐
│  实体C  │
└─────────┘
```

### 3. 逻辑设计阶段

**任务**
- 将ER图转换为关系模型
- 确定表结构
- 定义主键和外键
- 应用规范化理论

**转换规则**
| ER元素 | 转换为 |
|--------|--------|
| 实体 | 表 |
| 属性 | 字段 |
| 多值属性 | 独立表 |
| 关系（1:1） | 外键或合并表 |
| 关系（1:N） | 外键在N端 |
| 关系（M:N） | 关联表 |

### 4. 物理设计阶段

**任务**
- 选择数据库类型（MySQL/PostgreSQL/Oracle等）
- 定义字段类型和长度
- 创建索引和约束
- 分区方案设计
- 存储参数配置

**字段类型选择原则**
- 满足业务需求的最小的类型
- 避免使用TEXT/BLOB存储大对象
- 日期使用DATE/DATETIME而非VARCHAR
- 金额使用DECIMAL而非FLOAT/DOUBLE

**表结构设计模板**
```sql
CREATE TABLE 表名 (
    -- 主键
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',

    -- 业务字段
    字段1 VARCHAR(50) NOT NULL COMMENT '字段说明',
    字段2 DECIMAL(10,2) DEFAULT 0 COMMENT '金额',

    -- 审计字段
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',

    -- 索引
    INDEX idx_字段1 (字段1),
    UNIQUE INDEX uk_字段2 (字段2)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='表注释';
```

## 最佳实践

### 1. 表设计规范

**命名规范**
- 表名：t_业务模块_实体名，如`t_user_info`、`t_order_detail`
- 字段名：下划线分隔，如`user_name`、`order_id`
- 索引名：`idx_` + 字段名，如`idx_user_name`
- 主键：统一使用`id`

**字段设计**
- 必须包含审计字段：created_at、updated_at、deleted_at
- 状态字段使用TINYINT或CHAR，定义状态值常量
- 预留字段避免使用，优先考虑扩展性设计
- 避免NULL字段，设置默认值

**表设计检查清单**
- [ ] 每张表有明确的主键
- [ ] 表名和字段名符合命名规范
- [ ] 字段类型和长度合理
- [ ] 必要的索引已创建
- [ ] 包含审计字段
- [ ] 添加了表和字段注释
- [ ] 删除了不必要的字段

### 2. 索引设计规范

**索引创建原则**
- 主键自动建立唯一索引
- 查询条件频繁的字段建立索引
- 排序、分组字段建立索引
- JOIN连接的字段建立索引
- 高选择性字段优先建索引

**复合索引设计**
- 遵循最左前缀原则
- 选择性高的字段放前面
- 考虑查询条件的组合
- 示例：查询`WHERE a=1 AND b=2 AND c=3`，索引顺序(a,b,c)

**索引维护**
- 定期分析索引使用情况
- 删除未使用的索引
- 监控索引碎片
- 大表考虑分区替代部分索引

### 3. SQL编写规范

**INSERT语句**
```sql
INSERT INTO t_user (name, email, status) VALUES ('张三', 'zhangsan@example.com', 1);
INSERT INTO t_user VALUES (1, '张三', 'zhangsan@example.com', 1);
INSERT INTO t_order (user_id, amount) SELECT user_id, SUM(amount) FROM t_order_temp GROUP BY user_id;
```

**UPDATE语句**
```sql
UPDATE t_user SET status = 2, updated_at = NOW() WHERE id = 1;
UPDATE t_user SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
```

**SELECT语句**
```sql
SELECT id, name, email FROM t_user WHERE status = 1 ORDER BY created_at DESC LIMIT 10;
SELECT a.id, a.name, b.order_count FROM t_user a LEFT JOIN (SELECT user_id, COUNT(*) as order_count FROM t_order GROUP BY user_id) b ON a.id = b.user_id;
```

**避免的写法**
```sql
SELECT * FROM t_user;
SELECT name, email FROM t_user WHERE id = '1';
SELECT * FROM t_order WHERE created_at LIKE '2024-01%';
```

### 4. 性能优化规范

**查询优化**
- 避免SELECT *
- 使用EXPLAIN分析查询计划
- 减少JOIN操作，优先在应用层处理
- 批量操作替代循环单条
- 合理使用LIMIT

**写入优化**
- 批量INSERT替代单条
- 异步写入非核心日志
- 避免大事务
- 使用延迟删除替代即时删除

**架构优化**
- 读写分离分散压力
- 分库分表处理大表
- 使用缓存减少数据库压力
- 历史数据归档

## 常见问题解决方案

### 1. 数据去重

**问题**：表中存在重复数据如何处理？

**解决方案**

方案一：保留一条，删除其余
```sql
DELETE FROM t_user WHERE id NOT IN (
    SELECT MIN(id) FROM t_user GROUP BY name, email
);
```

方案二：创建无重复表
```sql
CREATE TABLE t_user_new AS
SELECT * FROM t_user GROUP BY name, email;

RENAME TABLE t_user TO t_user_old, t_user_new TO t_user;
```

**预防措施**
- 建表时添加唯一约束
- 定期使用数据质量工具检测

### 2. 大表结构变更

**问题**：在线表添加字段、添加索引影响业务

**解决方案**

添加字段
```sql
ALTER TABLE t_order ADD COLUMN shipping_address VARCHAR(500) DEFAULT NULL COMMENT '收货地址',
ALGORITHM=INPLACE, LOCK=NONE;
```

添加索引
```sql
CREATE INDEX idx_user_id ON t_order(user_id), ALGORITHM=INPLACE, LOCK=NONE;
```

使用pt-online-schema-change
```bash
pt-online-schema-change --alter "ADD COLUMN new_col VARCHAR(100)" D=t_order,t=t_order
```

### 3. 分页查询性能

**问题**：深度分页查询越来越慢

**问题SQL**
```sql
SELECT * FROM t_order ORDER BY id LIMIT 1000000, 10;
```

**解决方案**

方案一：使用ID分页
```sql
SELECT * FROM t_order WHERE id > 1000000 ORDER BY id LIMIT 10;
```

方案二：使用延迟关联
```sql
SELECT a.* FROM t_order a
INNER JOIN (SELECT id FROM t_order ORDER BY id LIMIT 1000000, 10) b
ON a.id = b.id;
```

方案三：使用游标
```sql
SELECT * FROM t_order WHERE id > ? ORDER BY id LIMIT 10;
```

### 4. 多表关联查询优化

**问题**：多表JOIN查询性能差

**解决方案**

分析问题
```sql
EXPLAIN SELECT a.*, b.name FROM t_order a
JOIN t_user b ON a.user_id = b.id
WHERE b.status = 1;
```

优化策略
- 确保关联字段有索引
- 减少关联表数量
- 控制返回数据量
- 拆分为简单查询
- 使用应用层JOIN替代数据库JOIN

### 5. 数据迁移

**问题**：如何安全迁移数据到新表？

**解决方案**

完整迁移流程
```sql
-- 1. 创建新表结构
CREATE TABLE t_user_new LIKE t_user;

-- 2. 迁移数据
INSERT INTO t_user_new SELECT * FROM t_user;

-- 3. 验证数据一致性
SELECT COUNT(*) FROM t_user;
SELECT COUNT(*) FROM t_user_new;

-- 4. 原子切换
RENAME TABLE t_user TO t_user_old, t_user_new TO t_user;

-- 5. 观察一段时间后删除旧表
DROP TABLE t_user_old;
```

### 6. 数据库容量规划

**问题**：如何评估数据库存储容量？

**评估方法**

表容量估算
```
单表预估容量 = 预估行数 × (字段总长度 + 索引长度 + 开销)
```

考虑因素
- 业务增长趋势
- 数据保留周期
- 索引数量
- 数据库预留空间（20-30%）

**容量规划检查清单**
- [ ] 评估当前数据量
- [ ] 预测3年后数据量
- [ ] 计算存储空间需求
- [ ] 规划扩容策略
- [ ] 设置容量告警

## 数据库类型选择

### 关系型数据库

| 数据库 | 适用场景 | 特点 |
|--------|---------|------|
| MySQL | Web应用、中小型系统 | 轻量、开源生态好 |
| PostgreSQL | 复杂查询、企业级应用 | 功能丰富、扩展性强 |
| Oracle | 大型企业核心系统 | 稳定性高、功能全面 |
| SQL Server | Windows环境企业应用 | 微软生态、图形化管理 |

### NoSQL数据库

| 类型 | 代表产品 | 适用场景 |
|------|---------|---------|
| Key-Value | Redis、Memcached | 缓存、Session存储 |
| Document | MongoDB | 非结构化数据、内容管理 |
| Column | Cassandra、HBase | 时序数据、大数据存储 |
| Graph | Neo4j | 社交关系、知识图谱 |

### 选型建议

**OLTP系统**
- 首选：MySQL、PostgreSQL
- 考虑分库分表：ShardingSphere

**OLAP系统**
- 首选：ClickHouse、StarRocks
- 传统选择：GreenPlum、Apache Hive

**混合场景**
- TiDB：兼容MySQL的NewSQL
- CockroachDB：分布式SQL

## 参考资源

本技能的参考资料目录包含：
- `design-checklist.md` - 设计检查清单
- `sql-templates.md` - SQL模板库
- `naming-conventions.md` - 命名规范文档
- `performance-tuning.md` - 性能调优指南
- `case-studies/` - 案例分析目录

---

*本技能持续更新，如有疑问或建议请联系维护者。*
