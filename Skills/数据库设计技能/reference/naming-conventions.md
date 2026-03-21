# 数据库命名规范

## 通用原则

1. **可读性**：名称应易于理解，顾名知义
2. **简洁性**：在保证清晰的前提下尽量简短
3. **一致性**：同一项目内命名风格必须统一
4. **可排序**：关键信息放在前面，便于排序和查找
5. **无保留字**：避免使用数据库保留字

## 字符集规范

- 仅使用：字母（a-z、A-Z）、数字（0-9）、下划线（_）
- 必须以字母开头
- 不使用中文、特殊字符、空格
- 不使用连续下划线（如 `__`）

## 数据库对象命名

### 数据库

```
{project}_{env}

示例：
- mall_dev
- mall_test
- mall_prod
```

### 表

```
t_{module}_{entity}

命名规范：
- t: 表前缀（小写t）
- module: 业务模块名
- entity: 实体名称（使用单数形式）

常见示例：
- t_sys_user（系统模块-用户表）
- t_sys_role（系统模块-角色表）
- t_sys_permission（系统模块-权限表）
- t_biz_order（业务模块-订单表）
- t_biz_order_item（业务模块-订单明细表）
- t_biz_product（业务模块-商品表）
- t_biz_category（业务模块-分类表）
- t_uc_user_info（用户中心模块-用户信息表）
- t_log_operation（日志模块-操作日志表）
- t_config_system（配置模块-系统配置表）

特殊表命名：
- t_rel_xxx_yyy：关联表（如 t_rel_user_role）
- t_tmp_xxx：临时表
- t_bak_xxx：备份表
- t_his_xxx：历史表
```

### 字段

```
{entity}_{attribute}

命名规范：
- 使用下划线分隔
- 全部小写
- 简写可接受但不推荐

常见示例：
- user_id（用户ID）
- user_name（用户名）
- real_name（真实姓名）
- email（邮箱）
- phone（手机号）
- password（密码）
- status（状态）
- order_id（订单ID）
- order_no（订单号）
- amount（金额）
- quantity（数量）
- price（价格）
- created_at（创建时间）
- updated_at（更新时间）
- deleted_at（删除时间）

布尔字段命名：
- is_xxx：如 is_deleted、is_enabled、is_valid
- has_xxx：如 has_permission、has_child
- can_xxx：如 can_edit、can_delete

状态字段值定义：
- status：1-正常/启用，0-禁用/删除
- order_status：1-待支付，2-已支付，3-已发货，4-已完成，5-已取消
```

### 索引

```
{type}_{table}_{column(s)}

索引类型前缀：
- idx_：普通索引
- uk_：唯一索引
- pk_：主键索引
- fk_：外键索引
- un_：全文索引

常见示例：
- idx_user_id（用户ID索引）
- uk_order_no（订单号唯一索引）
- pk_id（主键索引）
- fk_order_user（订单表用户ID外键索引）
- idx_user_status_created（用户表状态和创建时间复合索引）

复合索引命名：
- idx_{table}_{col1}_{col2}_{col3}
- 字段按选择性和重要程度排序
```

### 外键

```
fk_{child_table}_{parent_table}_{column}

示例：
- fk_order_user_user_id（订单表用户ID外键）
- fk_order_item_order_order_id（订单明细表订单ID外键）
```

### 视图

```
v_{module}_{entity}

示例：
- v_sys_user_active（系统模块-活跃用户视图）
- v_biz_order_summary（业务模块-订单汇总视图）
```

### 存储过程

```
sp_{module}_{action}

示例：
- sp_user_reset_password（用户模块-重置密码）
- sp_order_settle（订单模块-结算）
```

### 触发器

```
tr_{table}_{action}

示例：
- tr_order_after_insert（订单表插入后触发）
- tr_user_before_update（用户表更新前触发）
```

### 函数

```
fn_{module}_{purpose}

示例：
- fn_order_calculate_amount（订单模块-计算金额）
```

## 字段类型命名

### 常用字段类型对照

| 业务含义 | MySQL类型 | 长度建议 | 说明 |
|---------|-----------|---------|------|
| 主键ID | BIGINT | - | 无符号，自增 |
| 状态 | TINYINT | - | 1字节，0-255 |
| 年龄 | TINYINT | - | 无符号0-255 |
| 数量 | INT | - | 通用整数 |
| 金额 | DECIMAL | (12,2) | 精确数值 |
| 比率 | DECIMAL | (5,4) | 如0.1234 |
| 体重/身高 | DECIMAL | (5,2) | 如99.99 |
| 电话 | VARCHAR | 20 | - |
| 手机号 | VARCHAR | 20 | 国内手机11位 |
| 邮编 | VARCHAR | 10 | - |
| 邮箱 | VARCHAR | 100 | - |
| 网址 | VARCHAR | 500 | - |
| IP地址 | VARCHAR | 50 | 支持IPv6 |
| 姓名 | VARCHAR | 50 | 中文姓名 |
| 标题 | VARCHAR | 200 | - |
| 描述 | VARCHAR | 500 | 短文本 |
| 简介 | TEXT | - | 长文本 |
| 内容 | TEXT/LONGTEXT | - | 富文本内容 |
| JSON | JSON | - | 结构化数据 |
| 日期 | DATE | - | YYYY-MM-DD |
| 时间 | DATETIME | - | YYYY-MM-DD HH:MM:SS |
| 时间戳 | TIMESTAMP | - | 自动更新 |

## 常量值命名

```sql
-- 状态值定义
CREATE TABLE t_sys_dict (
    id BIGINT NOT NULL AUTO_INCREMENT,
    type VARCHAR(50) NOT NULL COMMENT '字典类型',
    code VARCHAR(20) NOT NULL COMMENT '字典码',
    value VARCHAR(100) NOT NULL COMMENT '字典值',
    sort INT DEFAULT 0 COMMENT '排序',
    status TINYINT DEFAULT 1 COMMENT '状态',
    PRIMARY KEY (id),
    UNIQUE KEY uk_type_code (type, code)
) COMMENT='字典表';

-- 常用字典类型
-- user_status: 用户状态
-- order_status: 订单状态
-- payment_method: 支付方式
-- shipping_method: 配送方式
```

## 大小写规范

### MySQL设置

```sql
-- Linux下MySQL区分大小写，建议统一小写
-- Windows下MySQL不区分大小写
-- 统一规范：所有名称使用小写
```

### 推荐写法

```sql
CREATE TABLE t_biz_order (
    id BIGINT NOT NULL AUTO_INCREMENT,
    order_no VARCHAR(32) NOT NULL,
    user_id BIGINT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    KEY idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

### 不推荐写法

```sql
-- 大写表名
CREATE TABLE T_BIZ_ORDER (...);

-- 混合大小写
CREATE TABLE t_Biz_Order (...);

-- 使用驼峰命名
CREATE TABLE tBizOrder (...);
```

## 命名长度限制

| 数据库对象 | 最大长度 | 推荐最大长度 |
|-----------|---------|-------------|
| 数据库名 | 64 | 30 |
| 表名 | 64 | 30 |
| 列名 | 64 | 30 |
| 索引名 | 64 | 30 |
| 视图名 | 64 | 30 |
| 存储过程名 | 64 | 30 |

## 命名示例

### 电商系统

```
t_sys_user          -- 系统用户
t_sys_role          -- 系统角色
t_sys_permission    -- 系统权限
t_sys_user_role     -- 用户角色关联
t_biz_category      -- 商品分类
t_biz_product       -- 商品
t_biz_sku           -- 商品SKU
t_biz_spec          -- 商品规格
t_biz_order         -- 订单
t_biz_order_item    -- 订单明细
t_biz_cart          -- 购物车
t_biz_address       -- 收货地址
t_pay_order         -- 支付订单
t_log_action        -- 操作日志
t_log_login         -- 登录日志
```

### 内容管理系统

```
t_cms_article       -- 文章
t_cms_category      -- 分类
t_cms_tag           -- 标签
t_cms_comment        -- 评论
t_cms_attachment    -- 附件
t_cms_link          -- 友情链接
```
