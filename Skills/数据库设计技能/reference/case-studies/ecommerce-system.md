# 电商系统数据库设计案例

## 系统概述

某中型电商平台，需要支持用户管理、商品管理、订单管理、支付管理、库存管理等核心业务。

## 核心表结构设计

### 用户模块

```sql
-- 用户基础信息表
CREATE TABLE t_sys_user (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(255) NOT NULL COMMENT '密码（加密存储）',
    email VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
    phone VARCHAR(20) DEFAULT NULL COMMENT '手机号',
    avatar_url VARCHAR(500) DEFAULT NULL COMMENT '头像',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-正常',
    user_type TINYINT NOT NULL DEFAULT 1 COMMENT '类型：1-普通用户，2-商家用户',
    last_login_at DATETIME DEFAULT NULL COMMENT '最后登录时间',
    last_login_ip VARCHAR(50) DEFAULT NULL COMMENT '最后登录IP',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_username (username),
    UNIQUE KEY uk_phone (phone),
    UNIQUE KEY uk_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- 用户地址表
CREATE TABLE t_uc_user_address (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '地址ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    receiver_name VARCHAR(100) NOT NULL COMMENT '收货人姓名',
    receiver_phone VARCHAR(20) NOT NULL COMMENT '收货人电话',
    province VARCHAR(50) NOT NULL COMMENT '省份',
    city VARCHAR(50) NOT NULL COMMENT '城市',
    district VARCHAR(50) NOT NULL COMMENT '区县',
    detail_address VARCHAR(255) NOT NULL COMMENT '详细地址',
    postal_code VARCHAR(10) DEFAULT NULL COMMENT '邮编',
    is_default TINYINT NOT NULL DEFAULT 0 COMMENT '是否默认：0-否，1-是',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    KEY idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户地址表';

-- 用户积分表
CREATE TABLE t_uc_user_point (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '记录ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    point_type TINYINT NOT NULL COMMENT '类型：1-获得，2-消费',
    point_amount INT NOT NULL COMMENT '积分数量',
    balance INT NOT NULL COMMENT '余额',
    source VARCHAR(50) NOT NULL COMMENT '来源：order/revoke/gift',
    source_id VARCHAR(64) DEFAULT NULL COMMENT '来源ID（如订单号）',
    remark VARCHAR(255) DEFAULT NULL COMMENT '备注',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    KEY idx_user_id (user_id),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户积分表';
```

### 商品模块

```sql
-- 商品分类表
CREATE TABLE t_biz_category (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '分类ID',
    parent_id BIGINT DEFAULT NULL COMMENT '父分类ID',
    category_name VARCHAR(100) NOT NULL COMMENT '分类名称',
    category_level TINYINT NOT NULL DEFAULT 1 COMMENT '层级：1-一级，2-二级，3-三级',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    KEY idx_parent_id (parent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品分类表';

-- 品牌表
CREATE TABLE t_biz_brand (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '品牌ID',
    brand_name VARCHAR(100) NOT NULL COMMENT '品牌名称',
    brand_logo VARCHAR(500) DEFAULT NULL COMMENT '品牌Logo',
    company_name VARCHAR(200) DEFAULT NULL COMMENT '公司名称',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='品牌表';

-- 商品表（SPU）
CREATE TABLE t_biz_product (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '商品ID',
    product_name VARCHAR(200) NOT NULL COMMENT '商品名称',
    category_id BIGINT NOT NULL COMMENT '分类ID',
    brand_id BIGINT DEFAULT NULL COMMENT '品牌ID',
    product_desc TEXT DEFAULT NULL COMMENT '商品描述',
    keywords VARCHAR(255) DEFAULT NULL COMMENT '关键词',
    main_image VARCHAR(500) NOT NULL COMMENT '主图',
    images TEXT DEFAULT NULL COMMENT '图片列表（JSON）',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-下架，1-上架，2-待审核',
    sales_count INT NOT NULL DEFAULT 0 COMMENT '销量',
    view_count INT NOT NULL DEFAULT 0 COMMENT '浏览量',
    review_count INT NOT NULL DEFAULT 0 COMMENT '评价数',
    avg_score DECIMAL(3,2) DEFAULT NULL COMMENT '平均评分',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    KEY idx_category_id (category_id),
    KEY idx_brand_id (brand_id),
    KEY idx_status (status),
    KEY idx_sales_count (sales_count)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品表（SPU）';

-- 商品SKU表
CREATE TABLE t_biz_sku (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'SKU ID',
    product_id BIGINT NOT NULL COMMENT '商品ID',
    sku_code VARCHAR(64) NOT NULL COMMENT 'SKU编码',
    sku_name VARCHAR(255) NOT NULL COMMENT 'SKU名称（包含规格）',
    price DECIMAL(12,2) NOT NULL COMMENT '售价',
    cost_price DECIMAL(12,2) DEFAULT NULL COMMENT '成本价',
    stock_quantity INT NOT NULL DEFAULT 0 COMMENT '库存数量',
    low_stock_threshold INT NOT NULL DEFAULT 10 COMMENT '低库存阈值',
    weight DECIMAL(10,2) DEFAULT NULL COMMENT '重量（克）',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-下架，1-上架',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_sku_code (sku_code),
    KEY idx_product_id (product_id),
    KEY idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品SKU表';

-- 商品规格表
CREATE TABLE t_biz_spec (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '规格ID',
    spec_name VARCHAR(50) NOT NULL COMMENT '规格名称（如颜色、尺寸）',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品规格表';

-- 规格值表
CREATE TABLE t_biz_spec_value (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '规格值ID',
    spec_id BIGINT NOT NULL COMMENT '规格ID',
    spec_value VARCHAR(100) NOT NULL COMMENT '规格值',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    KEY idx_spec_id (spec_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='规格值表';

-- SKU与规格关联表
CREATE TABLE t_biz_sku_spec (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    sku_id BIGINT NOT NULL COMMENT 'SKU ID',
    spec_id BIGINT NOT NULL COMMENT '规格ID',
    spec_value_id BIGINT NOT NULL COMMENT '规格值ID',
    PRIMARY KEY (id),
    KEY idx_sku_id (sku_id),
    KEY idx_spec_id (spec_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='SKU规格关联表';
```

### 订单模块

```sql
-- 订单表
CREATE TABLE t_biz_order (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '订单ID',
    order_no VARCHAR(32) NOT NULL COMMENT '订单号',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    total_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '商品总金额',
    discount_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '优惠金额',
    freight_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '运费',
    pay_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '应付金额',
    point_discount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '积分抵扣金额',
    coupon_discount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '优惠券抵扣',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：1-待支付，2-已支付，3-待发货，4-已发货，5-待收货，6-已完成，7-已取消，8-退款中，9-已退款',
    address_id BIGINT NOT NULL COMMENT '收货地址ID',
    receiver_name VARCHAR(100) NOT NULL COMMENT '收货人',
    receiver_phone VARCHAR(20) NOT NULL COMMENT '联系电话',
    province VARCHAR(50) NOT NULL COMMENT '省份',
    city VARCHAR(50) NOT NULL COMMENT '城市',
    district VARCHAR(50) NOT NULL COMMENT '区县',
    detail_address VARCHAR(255) NOT NULL COMMENT '详细地址',
    buyer_remark VARCHAR(255) DEFAULT NULL COMMENT '买家备注',
    pay_time DATETIME DEFAULT NULL COMMENT '支付时间',
    ship_time DATETIME DEFAULT NULL COMMENT '发货时间',
    receive_time DATETIME DEFAULT NULL COMMENT '收货时间',
    cancel_time DATETIME DEFAULT NULL COMMENT '取消时间',
    cancel_reason VARCHAR(255) DEFAULT NULL COMMENT '取消原因',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '下单时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_order_no (order_no),
    KEY idx_user_id (user_id),
    KEY idx_status (status),
    KEY idx_created_at (created_at),
    KEY idx_pay_time (pay_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';

-- 订单明细表
CREATE TABLE t_biz_order_item (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '明细ID',
    order_id BIGINT NOT NULL COMMENT '订单ID',
    order_no VARCHAR(32) NOT NULL COMMENT '订单号',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    sku_id BIGINT NOT NULL COMMENT 'SKU ID',
    product_id BIGINT NOT NULL COMMENT '商品ID',
    product_name VARCHAR(200) NOT NULL COMMENT '商品名称',
    sku_name VARCHAR(255) NOT NULL COMMENT 'SKU规格名称',
    sku_image VARCHAR(500) DEFAULT NULL COMMENT 'SKU图片',
    price DECIMAL(12,2) NOT NULL COMMENT '单价',
    quantity INT NOT NULL COMMENT '数量',
    subtotal DECIMAL(12,2) NOT NULL COMMENT '小计金额',
    discount_amount DECIMAL(12,2) NOT NULL DEFAULT 0.00 COMMENT '优惠金额',
    point_amount INT NOT NULL DEFAULT 0 COMMENT '使用积分',
    item_status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：1-正常，2-退款中，3-已退款',
    refund_time DATETIME DEFAULT NULL COMMENT '退款时间',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    KEY idx_order_id (order_id),
    KEY idx_order_no (order_no),
    KEY idx_sku_id (sku_id),
    KEY idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单明细表';

-- 购物车表
CREATE TABLE t_biz_cart (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '购物车ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    sku_id BIGINT NOT NULL COMMENT 'SKU ID',
    quantity INT NOT NULL DEFAULT 1 COMMENT '数量',
    selected TINYINT NOT NULL DEFAULT 1 COMMENT '是否选中：0-未选中，1-选中',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '添加时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_user_sku (user_id, sku_id),
    KEY idx_user_id (user_id),
    KEY idx_sku_id (sku_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='购物车表';
```

### 库存模块

```sql
-- 库存表
CREATE TABLE t_stock_sku (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    sku_id BIGINT NOT NULL COMMENT 'SKU ID',
    warehouse_id BIGINT NOT NULL COMMENT '仓库ID',
    available_quantity INT NOT NULL DEFAULT 0 COMMENT '可用库存',
    locked_quantity INT NOT NULL DEFAULT 0 COMMENT '锁定库存',
    occupied_quantity INT NOT NULL DEFAULT 0 COMMENT '占用库存（在途订单）',
    safety_stock INT NOT NULL DEFAULT 0 COMMENT '安全库存',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_sku_warehouse (sku_id, warehouse_id),
    KEY idx_sku_id (sku_id),
    KEY idx_warehouse_id (warehouse_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存表';

-- 库存流水表
CREATE TABLE t_stock_flow (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '流水ID',
    sku_id BIGINT NOT NULL COMMENT 'SKU ID',
    warehouse_id BIGINT NOT NULL COMMENT '仓库ID',
    flow_type TINYINT NOT NULL COMMENT '类型：1-入库，2-出库，3-锁定，4-解锁',
    quantity INT NOT NULL COMMENT '数量（正负）',
    before_quantity INT NOT NULL COMMENT '变动前数量',
    after_quantity INT NOT NULL COMMENT '变动后数量',
    order_no VARCHAR(64) DEFAULT NULL COMMENT '关联订单号',
    source VARCHAR(50) NOT NULL COMMENT '来源：purchase/order_return/adjustment',
    source_id VARCHAR(64) DEFAULT NULL COMMENT '来源ID',
    remark VARCHAR(255) DEFAULT NULL COMMENT '备注',
    operator_id BIGINT DEFAULT NULL COMMENT '操作人ID',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '操作时间',
    PRIMARY KEY (id),
    KEY idx_sku_id (sku_id),
    KEY idx_order_no (order_no),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存流水表';
```

## 设计要点总结

### 1. 订单号生成策略

```sql
-- 订单号格式：时间戳(14位) + 随机数(6位) + 用户ID后4位
-- 示例：2024011514302500001234

-- 生成订单号存储过程
DELIMITER //
CREATE DEFINER=`root`@`%` PROCEDURE `generate_order_no`(OUT order_no VARCHAR(32))
BEGIN
    DECLARE random_str VARCHAR(6);
    SET random_str = LPAD(FLOOR(RAND() * 1000000), 6, '0');
    SET order_no = CONCAT(DATE_FORMAT(NOW(), '%Y%m%d%H%i%s'), random_str);
END //
DELIMITER ;
```

### 2. 库存扣减实现

```sql
-- 乐观锁扣减库存
UPDATE t_stock_sku
SET available_quantity = available_quantity - #{quantity},
    locked_quantity = locked_quantity + #{quantity},
    updated_at = NOW()
WHERE sku_id = #{sku_id}
AND warehouse_id = #{warehouse_id}
AND available_quantity >= #{quantity};

-- 检查影响行数，如果为0则库存不足
```

### 3. 订单状态流转

```
待支付 → 已支付 → 待发货 → 已发货 → 待收货 → 已完成
   ↓         ↓        ↓
已取消   已退款    退款中 → 已退款
```

### 4. 分库分表策略

```sql
-- 按用户ID分库分表（ShardingSphere示例）
sharding.rule:
  tables:
    t_biz_order:
      actualDataNodes: ds_${0..1}.t_biz_order_${0..15}
      databaseStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: database_mod
      tableStrategy:
        standard:
          shardingColumn: user_id
          shardingAlgorithmName: table_mod
      keyGenerateStrategy:
        column: id
        keyGeneratorName: snowflake
```
