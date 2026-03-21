# 内容管理系统数据库设计案例

## 系统概述

某内容管理平台，支持文章发布、分类管理、标签管理、评论管理、用户管理、权限管理等核心功能。

## 核心表结构设计

### 用户权限模块

```sql
-- 用户表
CREATE TABLE t_sys_user (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL COMMENT '用户名',
    password VARCHAR(255) NOT NULL COMMENT '密码',
    real_name VARCHAR(100) DEFAULT NULL COMMENT '真实姓名',
    email VARCHAR(100) DEFAULT NULL COMMENT '邮箱',
    phone VARCHAR(20) DEFAULT NULL COMMENT '手机号',
    avatar_url VARCHAR(500) DEFAULT NULL COMMENT '头像',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-正常',
    user_type TINYINT NOT NULL DEFAULT 1 COMMENT '类型：1-普通用户，2-管理员',
    last_login_at DATETIME DEFAULT NULL COMMENT '最后登录时间',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_username (username),
    UNIQUE KEY uk_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- 角色表
CREATE TABLE t_sys_role (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '角色ID',
    role_code VARCHAR(50) NOT NULL COMMENT '角色代码',
    role_name VARCHAR(100) NOT NULL COMMENT '角色名称',
    description VARCHAR(255) DEFAULT NULL COMMENT '描述',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-禁用，1-启用',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_role_code (role_code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='角色表';

-- 权限表
CREATE TABLE t_sys_permission (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '权限ID',
    parent_id BIGINT DEFAULT NULL COMMENT '父权限ID',
    permission_code VARCHAR(100) NOT NULL COMMENT '权限代码',
    permission_name VARCHAR(100) NOT NULL COMMENT '权限名称',
    permission_type TINYINT NOT NULL COMMENT '类型：1-菜单，2-按钮，3-接口',
    path VARCHAR(255) DEFAULT NULL COMMENT '路径',
    icon VARCHAR(100) DEFAULT NULL COMMENT '图标',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_permission_code (permission_code),
    KEY idx_parent_id (parent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='权限表';

-- 用户角色关联表
CREATE TABLE t_sys_user_role (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    role_id BIGINT NOT NULL COMMENT '角色ID',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_user_role (user_id, role_id),
    KEY idx_role_id (role_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户角色关联表';

-- 角色权限关联表
CREATE TABLE t_sys_role_permission (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    role_id BIGINT NOT NULL COMMENT '角色ID',
    permission_id BIGINT NOT NULL COMMENT '权限ID',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_role_permission (role_id, permission_id),
    KEY idx_permission_id (permission_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='角色权限关联表';
```

### 内容管理模块

```sql
-- 文章分类表
CREATE TABLE t_cms_category (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '分类ID',
    parent_id BIGINT DEFAULT NULL COMMENT '父分类ID',
    category_name VARCHAR(100) NOT NULL COMMENT '分类名称',
    slug VARCHAR(100) NOT NULL COMMENT '别名（URL友好）',
    description VARCHAR(500) DEFAULT NULL COMMENT '描述',
    cover_image VARCHAR(500) DEFAULT NULL COMMENT '封面图',
    category_level TINYINT NOT NULL DEFAULT 1 COMMENT '层级',
    sort_order INT NOT NULL DEFAULT 0 COMMENT '排序',
    seo_title VARCHAR(200) DEFAULT NULL COMMENT 'SEO标题',
    seo_keywords VARCHAR(255) DEFAULT NULL COMMENT 'SEO关键词',
    seo_description VARCHAR(500) DEFAULT NULL COMMENT 'SEO描述',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-隐藏，1-显示',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_slug (slug),
    KEY idx_parent_id (parent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章分类表';

-- 标签表
CREATE TABLE t_cms_tag (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '标签ID',
    tag_name VARCHAR(50) NOT NULL COMMENT '标签名称',
    tag_slug VARCHAR(50) NOT NULL COMMENT '标签别名',
    article_count INT NOT NULL DEFAULT 0 COMMENT '文章数量',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_tag_slug (tag_slug)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='标签表';

-- 文章表
CREATE TABLE t_cms_article (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '文章ID',
    title VARCHAR(200) NOT NULL COMMENT '标题',
    slug VARCHAR(200) NOT NULL COMMENT '别名',
    summary TEXT DEFAULT NULL COMMENT '摘要',
    content LONGTEXT NOT NULL COMMENT '正文内容',
    cover_image VARCHAR(500) DEFAULT NULL COMMENT '封面图',
    author_id BIGINT NOT NULL COMMENT '作者ID',
    author_name VARCHAR(100) NOT NULL COMMENT '作者名称',
    category_id BIGINT DEFAULT NULL COMMENT '分类ID',
    view_count INT NOT NULL DEFAULT 0 COMMENT '浏览量',
    like_count INT NOT NULL DEFAULT 0 COMMENT '点赞数',
    comment_count INT NOT NULL DEFAULT 0 COMMENT '评论数',
    collect_count INT NOT NULL DEFAULT 0 COMMENT '收藏数',
    is_top TINYINT NOT NULL DEFAULT 0 COMMENT '是否置顶：0-否，1-是',
    is_recommend TINYINT NOT NULL DEFAULT 0 COMMENT '是否推荐：0-否，1-是',
    is_original TINYINT NOT NULL DEFAULT 1 COMMENT '是否原创：0-转载，1-原创',
    source_url VARCHAR(500) DEFAULT NULL COMMENT '转载来源',
    seo_title VARCHAR(200) DEFAULT NULL COMMENT 'SEO标题',
    seo_keywords VARCHAR(255) DEFAULT NULL COMMENT 'SEO关键词',
    seo_description VARCHAR(500) DEFAULT NULL COMMENT 'SEO描述',
    publish_status TINYINT NOT NULL DEFAULT 0 COMMENT '发布状态：0-草稿，1-已发布，2-待审核',
    publish_time DATETIME DEFAULT NULL COMMENT '发布时间',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-下架，1-上架',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at DATETIME DEFAULT NULL COMMENT '删除时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_slug (slug),
    KEY idx_author_id (author_id),
    KEY idx_category_id (category_id),
    KEY idx_publish_time (publish_time),
    KEY idx_status (status),
    KEY idx_is_top (is_top),
    KEY idx_is_recommend (is_recommend)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章表';

-- 文章标签关联表
CREATE TABLE t_cms_article_tag (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    article_id BIGINT NOT NULL COMMENT '文章ID',
    tag_id BIGINT NOT NULL COMMENT '标签ID',
    PRIMARY KEY (id),
    UNIQUE KEY uk_article_tag (article_id, tag_id),
    KEY idx_tag_id (tag_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章标签关联表';

-- 评论表
CREATE TABLE t_cms_comment (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '评论ID',
    article_id BIGINT NOT NULL COMMENT '文章ID',
    parent_id BIGINT DEFAULT NULL COMMENT '父评论ID（回复）',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    user_name VARCHAR(100) NOT NULL COMMENT '用户名',
    user_avatar VARCHAR(500) DEFAULT NULL COMMENT '用户头像',
    content TEXT NOT NULL COMMENT '评论内容',
    like_count INT NOT NULL DEFAULT 0 COMMENT '点赞数',
    reply_count INT NOT NULL DEFAULT 0 COMMENT '回复数',
    ip_address VARCHAR(50) DEFAULT NULL COMMENT 'IP地址',
    user_agent VARCHAR(500) DEFAULT NULL COMMENT 'User Agent',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态：0-隐藏，1-显示',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    PRIMARY KEY (id),
    KEY idx_article_id (article_id),
    KEY idx_user_id (user_id),
    KEY idx_parent_id (parent_id),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论表';

-- 附件表
CREATE TABLE t_cms_attachment (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '附件ID',
    file_name VARCHAR(255) NOT NULL COMMENT '文件名',
    file_path VARCHAR(500) NOT NULL COMMENT '文件路径',
    file_url VARCHAR(500) NOT NULL COMMENT '访问URL',
    file_type VARCHAR(50) NOT NULL COMMENT '文件类型',
    file_size BIGINT NOT NULL COMMENT '文件大小（字节）',
    mime_type VARCHAR(100) NOT NULL COMMENT 'MIME类型',
    uploader_id BIGINT NOT NULL COMMENT '上传者ID',
    uploader_name VARCHAR(100) NOT NULL COMMENT '上传者名称',
    download_count INT NOT NULL DEFAULT 0 COMMENT '下载次数',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '上传时间',
    PRIMARY KEY (id),
    KEY idx_uploader_id (uploader_id),
    KEY idx_file_type (file_type),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='附件表';
```

### 消息通知模块

```sql
-- 消息通知表
CREATE TABLE t_sys_notification (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '消息ID',
    user_id BIGINT NOT NULL COMMENT '接收用户ID',
    title VARCHAR(200) NOT NULL COMMENT '标题',
    content TEXT DEFAULT NULL COMMENT '内容',
    notification_type VARCHAR(50) NOT NULL COMMENT '类型：system/comment/like/follow',
    source_type VARCHAR(50) DEFAULT NULL COMMENT '来源类型：article/comment',
    source_id BIGINT DEFAULT NULL COMMENT '来源ID',
    is_read TINYINT NOT NULL DEFAULT 0 COMMENT '是否已读：0-未读，1-已读',
    read_time DATETIME DEFAULT NULL COMMENT '阅读时间',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    PRIMARY KEY (id),
    KEY idx_user_id (user_id),
    KEY idx_is_read (is_read),
    KEY idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='消息通知表';

-- 用户收藏表
CREATE TABLE t_uc_collect (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '收藏ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    collect_type VARCHAR(50) NOT NULL COMMENT '收藏类型：article/product',
    source_id BIGINT NOT NULL COMMENT '来源ID',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '收藏时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_user_source (user_id, collect_type, source_id),
    KEY idx_user_id (user_id),
    KEY idx_source_id (source_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户收藏表';

-- 用户点赞表
CREATE TABLE t_uc_like (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT '点赞ID',
    user_id BIGINT NOT NULL COMMENT '用户ID',
    like_type VARCHAR(50) NOT NULL COMMENT '点赞类型：article/comment',
    source_id BIGINT NOT NULL COMMENT '来源ID',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '点赞时间',
    PRIMARY KEY (id),
    UNIQUE KEY uk_user_source (user_id, like_type, source_id),
    KEY idx_user_id (user_id),
    KEY idx_source_id (source_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户点赞表';
```

## 设计要点总结

### 1. 内容审核流程

```sql
-- 文章状态流转
草稿 → 待审核 → 已发布 → 已下线
                 ↓
              审核拒绝 → 返回草稿

-- 审核记录表
CREATE TABLE t_cms_article_audit (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    article_id BIGINT NOT NULL COMMENT '文章ID',
    audit_action TINYINT NOT NULL COMMENT '动作：1-提交审核，2-通过，3-拒绝',
    audit_remark VARCHAR(255) DEFAULT NULL COMMENT '审核备注',
    auditor_id BIGINT DEFAULT NULL COMMENT '审核人ID',
    auditor_name VARCHAR(100) DEFAULT NULL COMMENT '审核人名称',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '审核时间',
    PRIMARY KEY (id),
    KEY idx_article_id (article_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章审核记录表';
```

### 2. 评论树形结构实现

```sql
-- 使用 parent_id 实现评论树
-- 对于深度较深的回复，使用闭合表（Closure Table）优化查询

-- 评论闭合表
CREATE TABLE t_cms_comment_closure (
    id BIGINT NOT NULL AUTO_INCREMENT COMMENT 'ID',
    ancestor_id BIGINT NOT NULL COMMENT '祖先评论ID',
    descendant_id BIGINT NOT NULL COMMENT '后代评论ID',
    depth INT NOT NULL COMMENT '层级深度',
    PRIMARY KEY (id),
    UNIQUE KEY uk_ancestor_descendant (ancestor_id, descendant_id),
    KEY idx_descendant_id (descendant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论闭合表';

-- 查询某评论的所有后代
SELECT c.* FROM t_cms_comment c
INNER JOIN t_cms_comment_closure cc ON c.id = cc.descendant_id
WHERE cc.ancestor_id = #{comment_id}
ORDER BY cc.depth, c.created_at;
```

### 3. 全文搜索配置

```sql
-- MySQL全文索引
ALTER TABLE t_cms_article ADD FULLTEXT INDEX ft_title_content (title, summary, content);

-- 查询示例
SELECT * FROM t_cms_article
WHERE MATCH(title, summary, content) AGAINST('关键词' IN NATURAL LANGUAGE MODE);

-- 考虑使用 Elasticsearch 作为搜索方案
```

### 4. SEO友好URL设计

```sql
-- 使用 slug 字段存储URL友好的别名
-- 格式：分类别名/文章ID-标题别名
-- 示例：tech/123-mysql-optimization-guide

-- 生成slug的存储过程
DELIMITER //
CREATE DEFINER=`root`@`%` PROCEDURE `generate_slug`(IN title VARCHAR(200), IN id BIGINT, OUT slug VARCHAR(200))
BEGIN
    DECLARE base_slug VARCHAR(200);
    SET base_slug = LOWER(REPLACE(title, ' ', '-'));
    SET base_slug = REGEXP_REPLACE(base_slug, '[^a-z0-9-]', '');
    SET base_slug = SUBSTRING(base_slug, 1, 100);
    SET slug = CONCAT(base_slug, '-', id);
END //
DELIMITER ;
```
