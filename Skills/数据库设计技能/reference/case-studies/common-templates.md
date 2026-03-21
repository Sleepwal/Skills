# 通用数据字典设计模板

## 系统配置表

```sql
CREATE TABLE t_sys_config (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '配置ID',
    config_key VARCHAR(100) NOT NULL COMMENT '配置键',
    config_value TEXT DEFAULT NULL COMMENT '配置值',
    config_type VARCHAR(50) NOT NULL COMMENT '类型：string/number/boolean/json/text',
    config_name VARCHAR(100) NOT NULL COMMENT '配置名称',
    config_group VARCHAR(50) NOT NULL DEFAULT 'default' COMMENT '配置分组',
    description VARCHAR(255) DEFAULT NULL COMMENT '描述',
    is_system TINYINT NOT NULL DEFAULT 0 COMMENT '是否系统配置：0-否，1-是',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_config_key (config_key)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统配置表';

-- 常用系统配置
INSERT INTO t_sys_config (config_key, config_value, config_type, config_name, config_group) VALUES
('site_name', '我的网站', 'string', '网站名称', 'basic'),
('site_logo', '/images/logo.png', 'string', '网站Logo', 'basic'),
('site_url', 'https://example.com', 'string', '网站地址', 'basic'),
('contact_email', 'admin@example.com', 'string', '联系邮箱', 'basic'),
('icp_number', '京ICP备12345678号', 'string', 'ICP备案号', 'basic'),
('copyright', '© 2024 公司名称 版权所有', 'string', '版权信息', 'basic'),
('items_per_page', '20', 'number', '每页条数', 'display'),
('allowed_file_types', 'jpg,jpeg,png,gif,pdf,doc,docx', 'string', '允许上传的文件类型', 'upload'),
('max_file_size', '10485760', 'number', '最大文件大小（字节）', 'upload'),
('image_max_width', '1920', 'number', '图片最大宽度', 'upload'),
('email_smtp_host', 'smtp.example.com', 'string', 'SMTP服务器', 'email'),
('email_smtp_port', '465', 'number', 'SMTP端口', 'email'),
('email_username', 'noreply@example.com', 'string', '邮箱用户名', 'email'),
('email_from_name', '网站管理员', 'string', '发件人名称', 'email'),
('jwt_secret', 'your-secret-key', 'string', 'JWT密钥', 'security'),
('jwt_expire_time', '86400', 'number', 'JWT过期时间（秒）', 'security'),
('password_min_length', '6', 'number', '密码最小长度', 'security'),
('login_max_attempts', '5', 'number', '登录最大尝试次数', 'security'),
('login_lock_time', '1800', 'number', '登录锁定时间（秒）', 'security');
```

## 通用字典表

```sql
CREATE TABLE t_sys_dict (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '字典ID',
    dict_type VARCHAR(50) NOT NULL COMMENT '字典类型',
    dict_code VARCHAR(50) NOT NULL COMMENT '字典码',
    dict_value VARCHAR(100) NOT NULL COMMENT '字典值',
    dict_label VARCHAR(100) NOT NULL COMMENT '字典标签',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    is_default TINYINT NOT NULL DEFAULT 0 COMMENT '是否默认：0-否，1-是',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_type_code (dict_type, dict_code),
    KEY idx_dict_type (dict_type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='字典表';

-- 用户状态
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('user_status', '0', '禁用', '禁用', 1),
('user_status', '1', '正常', '正常', 2);

-- 用户类型
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('user_type', '1', '普通用户', '普通用户', 1),
('user_type', '2', 'VIP用户', 'VIP用户', 2),
('user_type', '3', '管理员', '管理员', 3);

-- 性别
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('gender', '0', '未知', '未知', 0),
('gender', '1', '男', '男', 1),
('gender', '2', '女', '女', 2);

-- 是/否
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('yes_no', '0', '否', '否', 0),
('yes_no', '1', '是', '是', 1);

-- 订单状态
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('order_status', '1', 'pending_payment', '待支付', 1),
('order_status', '2', 'paid', '已支付', 2),
('order_status', '3', 'pending_ship', '待发货', 3),
('order_status', '4', 'shipped', '已发货', 4),
('order_status', '5', 'pending_receive', '待收货', 5),
('order_status', '6', 'completed', '已完成', 6),
('order_status', '7', 'cancelled', '已取消', 7),
('order_status', '8', 'refunding', '退款中', 8),
('order_status', '9', 'refunded', '已退款', 9);

-- 支付方式
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('payment_method', '1', 'wechat', '微信支付', 1),
('payment_method', '2', 'alipay', '支付宝', 2),
('payment_method', '3', 'unionpay', '银联支付', 3),
('payment_method', '4', 'balance', '余额支付', 4);

-- 支付状态
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('payment_status', '0', 'unpaid', '未支付', 0),
('payment_status', '1', 'paid', '已支付', 1),
('payment_status', '2', 'refunded', '已退款', 2);

-- 发货方式
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('shipping_method', '1', 'express', '快递', 1),
('shipping_method', '2', 'self_pickup', '自提', 2),
('shipping_method', '3', 'virtual', '虚拟发货', 3);

-- 文章发布状态
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('publish_status', '0', 'draft', '草稿', 0),
('publish_status', '1', 'published', '已发布', 1),
('publish_status', '2', 'pending_review', '待审核', 2);

-- 文章审核状态
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('audit_status', '1', 'pending', '待审核', 1),
('audit_status', '2', 'approved', '审核通过', 2),
('audit_status', '3', 'rejected', '审核拒绝', 3);

-- 日志级别
INSERT INTO t_sys_dict (dict_type, dict_code, dict_value, dict_label, sort_order) VALUES
('log_level', '1', 'debug', '调试', 1),
('log_level', '2', 'info', '信息', 2),
('log_level', '3', 'warn', '警告', 3),
('log_level', '4', 'error', '错误', 4);
```

## 地区表

```sql
CREATE TABLE t_sys_region (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '地区ID',
    parent_id BIGINT DEFAULT NULL COMMENT '父地区ID',
    region_code VARCHAR(20) NOT NULL COMMENT '地区代码',
    region_name VARCHAR(100) NOT NULL COMMENT '地区名称',
    region_level TINYINT NOT NULL COMMENT '层级：1-省，2-市，3-区县',
    postal_code VARCHAR(10) DEFAULT NULL COMMENT '邮编',
    city_code VARCHAR(20) DEFAULT NULL COMMENT '城市代码',
    longitude DECIMAL(10,6) DEFAULT NULL COMMENT '经度',
    latitude DECIMAL(10,6) DEFAULT NULL COMMENT '纬度',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    KEY idx_parent_id (parent_id),
    KEY idx_region_code (region_code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='地区表';
```

## 操作日志表

```sql
CREATE TABLE t_log_operation (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '日志ID',
    user_id BIGINT DEFAULT NULL COMMENT '用户ID',
    username VARCHAR(50) DEFAULT NULL COMMENT '用户名',
    operation_module VARCHAR(50) NOT NULL COMMENT '操作模块',
    operation_type VARCHAR(50) NOT NULL COMMENT '操作类型',
    operation_desc VARCHAR(255) DEFAULT NULL COMMENT '操作描述',
    request_method VARCHAR(10) NOT NULL COMMENT '请求方法',
    request_url VARCHAR(500) NOT NULL COMMENT '请求URL',
    request_params TEXT DEFAULT NULL COMMENT '请求参数',
    request_body TEXT DEFAULT NULL COMMENT '请求体',
    response_data TEXT DEFAULT NULL COMMENT '响应数据',
    ip_address VARCHAR(50) DEFAULT NULL COMMENT 'IP地址',
    user_agent VARCHAR(500) DEFAULT NULL COMMENT 'User Agent',
    execute_time INT DEFAULT NULL COMMENT '执行时间（毫秒）',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：1-成功，0-失败',
    error_message TEXT DEFAULT NULL COMMENT '错误信息',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '操作时间',
    PRIMARY KEY (id),
    KEY idx_user_id (user_id),
    KEY idx_operation_module (operation_module),
    KEY idx_operation_type (operation_type),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='操作日志表';
```

## 登录日志表

```sql
CREATE TABLE t_log_login (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '日志ID',
    user_id BIGINT DEFAULT NULL COMMENT '用户ID',
    username VARCHAR(50) DEFAULT NULL COMMENT '用户名',
    login_type TINYINT NOT NULL COMMENT '登录类型：1-登录，2-登出',
    login_platform VARCHAR(50) DEFAULT NULL COMMENT '登录平台',
    ip_address VARCHAR(50) DEFAULT NULL COMMENT 'IP地址',
    country VARCHAR(50) DEFAULT NULL COMMENT '国家',
    province VARCHAR(50) DEFAULT NULL COMMENT '省份',
    city VARCHAR(50) DEFAULT NULL COMMENT '城市',
    device_type VARCHAR(50) DEFAULT NULL COMMENT '设备类型',
    browser_type VARCHAR(100) DEFAULT NULL COMMENT '浏览器类型',
    os_type VARCHAR(50) DEFAULT NULL COMMENT '操作系统',
    user_agent TEXT DEFAULT NULL COMMENT 'User Agent',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '登录状态：1-成功，0-失败',
    fail_reason VARCHAR(255) DEFAULT NULL COMMENT '失败原因',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '登录时间',
    PRIMARY KEY (id),
    KEY idx_user_id (user_id),
    KEY idx_username (username),
    KEY idx_ip_address (ip_address),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='登录日志表';
```

## 定时任务调度表

```sql
CREATE TABLE t_sys_job (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '任务ID',
    job_name VARCHAR(100) NOT NULL COMMENT '任务名称',
    job_group VARCHAR(100) NOT NULL DEFAULT 'default' COMMENT '任务分组',
    job_class VARCHAR(255) NOT NULL COMMENT '任务类名',
    cron_expression VARCHAR(100) DEFAULT NULL COMMENT 'CRON表达式',
    interval_value INT DEFAULT NULL COMMENT '间隔时间（秒）',
    is_concurrent TINYINT NOT NULL DEFAULT 1 COMMENT '是否并发：0-否，1-是',
    misfire_policy TINYINT NOT NULL DEFAULT 1 COMMENT '失火策略：1-立即执行，2-下次执行，3-不执行',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-暂停，1-运行中',
    last_run_time DATETIME DEFAULT NULL COMMENT '上次运行时间',
    next_run_time DATETIME DEFAULT NULL COMMENT '下次运行时间',
    run_times INT NOT NULL DEFAULT 0 COMMENT '累计运行次数',
    description VARCHAR(255) DEFAULT NULL COMMENT '描述',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_job_name_group (job_name, job_group)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='定时任务表';

CREATE TABLE t_sys_job_log (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '日志ID',
    job_id BIGINT NOT NULL COMMENT '任务ID',
    job_name VARCHAR(100) NOT NULL COMMENT '任务名称',
    job_group VARCHAR(100) NOT NULL COMMENT '任务分组',
    start_time DATETIME NOT NULL COMMENT '开始时间',
    end_time DATETIME DEFAULT NULL COMMENT '结束时间',
    execute_time INT DEFAULT NULL COMMENT '执行时间（毫秒）',
    status TINYINT NOT NULL COMMENT '状态：1-成功，0-失败',
    error_message TEXT DEFAULT NULL COMMENT '错误信息',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    KEY idx_job_id (job_id),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='任务执行日志表';
```

## 消息队列表

```sql
CREATE TABLE t_sys_message_queue (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '消息ID',
    message_key VARCHAR(100) NOT NULL COMMENT '消息Key',
    message_type VARCHAR(50) NOT NULL COMMENT '消息类型',
    message_body TEXT NOT NULL COMMENT '消息体',
    priority TINYINT NOT NULL DEFAULT 5 COMMENT '优先级：1-高，5-中，10-低',
    status TINYINT NOT NULL DEFAULT 0 COMMENT '状态：0-待处理，1-处理中，2-已处理，3-处理失败',
    retry_count INT NOT NULL DEFAULT 0 COMMENT '重试次数',
    max_retry INT NOT NULL DEFAULT 3 COMMENT '最大重试次数',
    process_time DATETIME DEFAULT NULL COMMENT '计划处理时间',
    start_process_time DATETIME DEFAULT NULL COMMENT '开始处理时间',
    end_process_time DATETIME DEFAULT NULL COMMENT '处理完成时间',
    error_message TEXT DEFAULT NULL COMMENT '错误信息',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    KEY idx_message_key (message_key),
    KEY idx_message_type (message_type),
    KEY idx_status (status),
    KEY idx_process_time (process_time)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='消息队列表';
```
