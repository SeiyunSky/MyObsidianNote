数据定义语言，定义数据库对象

```mysql
查询所有数据库
SHOW DATABASES()

查询当前数据库
SELECT DATABASE()

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
```

修改数据表中的字段
```mysql
ALTER TABLE 表名 ADD 字段名 类型(长度) [COMMENT 注释] [约束]

ALTER TABLE employees 
ADD phone_number VARCHAR(20) COMMENT '联系电话' NOT NULL;

删除
Alter TABLE 表名 DROP 字段名;

```