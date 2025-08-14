## Insert优化
```mysql
批量插入
insert into * values ()()

手动提交事务
start transaction;

commit;

主键顺序插入
```

- 大批量插入数据
```mysql
利用Load
# 客户端连接服务端时，加上参数--local-infile
mysql --local-infile -u root -p

# 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile=1;

# 执行load指令将准备好的数据，加载到表结构中
load data local infile '文件g' into table '表名' fields terminated by ',' lines terminated by '\n';
```

```java
// 使用 BatchExecutor 执行批量插入
try (SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH)) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    // 从文件读取数据
    BufferedReader reader = new BufferedReader(new FileReader("/root/sql1.log"));
    String line;
    while ((line = reader.readLine()) != null) {
        String[] parts = line.split(","); // 按逗号分隔
        User user = new User(parts[0], parts[1], Integer.parseInt(parts[2]));
        mapper.insertUser(user);
    }
    
    sqlSession.commit();
}
```

## 主键优化
- **数据组织方式**
    在InnoDB中，表数据都是根据主键顺序组织存放的，称为索引组织表
- 页分裂，页合并
- 尽量降低主键长度，不要用UUID或身份证号做主键

## **Order By**
- Using Filesort 全表扫描，而非通过索引查询，查完后在缓冲区完成排序
- Using index，用有序索引扫描并返回有序数据
- 尽量通过有序索引去进行查询，能够快速检索

## **Group By**
为 GROUP BY 列创建合适索引

## **Limit** 
LIMIT 子句常用于分页查询，但在大数据量场景下可能导致性能问题。
- 创建覆盖索引+子查询实现
```mysql
SELECT t.* FROM large_table t, ( SELECT id FROM large_table ORDER BY score DESC LIMIT 10 OFFSET 1000 ) temp where t.id = temp.id;
```

## **Count优化**
![[Pasted image 20250804212546.png]]
![[Pasted image 20250804212553.png]]


## **Update优化**
```mysql
-- 创建条件列索引
CREATE INDEX idx_user_status ON users(status);
-- 更新时将利用索引定位
UPDATE users SET last_login = NOW() WHERE status = 'active';

-- 添加LIMIT限制（MySQL语法）
UPDATE products SET stock = stock - 1 WHERE product_id = 100 LIMIT 1;
-- 分批次更新大数据量
UPDATE large_table SET flag = 1 WHERE id BETWEEN 1 AND 10000;
UPDATE large_table SET flag = 1 WHERE id BETWEEN 10001 AND 20000;
```