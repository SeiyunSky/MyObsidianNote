数据定义语言，定义数据库对象

```mysql
查询所有数据库
SHOW DATABASES();

SHOW CREATE TABLE 表名;

查询当前数据库
SELECT DATABASE();

创建数据库与表
CREATE DATABASE 数据库名 [字符集] [COLLATE 排序规则]
CREATE TABLE 表名(  
字段1 字段1类型 [COMMENT 字段1注释],  
字段2 字段2类型 [COMMENT 字段2注释],  
字段3 字段3类型 [COMMENT 字段3注释],  
...  
字段n 字段n类型 [COMMENT 字段n注释]  
)[COMMENT 表注释];
```
建表例子：
```mysql
CREATE TABLE tb_user(
    id INT COMMENT '编号',
    name VARCHAR(50) COMMENT '姓名',
    age INT COMMENT '年龄',
    gender VARCHAR(1) COMMENT '性别'
) COMMENT '用户表';

CREATE TABLE `t_link_os_stats` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `full_short_url` varchar(128) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '完整短链接',
  `gid` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL COMMENT '短链接分组标识',
  `date` date DEFAULT NULL,
  `cnt` int DEFAULT NULL COMMENT '访问量',
  `os` varchar(64) DEFAULT NULL COMMENT '操作系统',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `del_flag` tinyint(1) DEFAULT NULL COMMENT '删除标识',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_full_short_url_date_gid` (`full_short_url`,`gid`,`date`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

修改数据表中的字段
```mysql
ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束]

ALTER TABLE employees 
ADD phone_number VARCHAR(20) COMMENT '联系电话' NOT NULL;

删除
Alter TABLE 表名 DROP 字段名;

```