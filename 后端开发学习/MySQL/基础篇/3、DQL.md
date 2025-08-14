## 查询语句
```MYSQL
select
    字段列表
from
    表名列表
where
    条件列表
group by
    分组字段列表（要包含select的全部字段）
having
    分组后条件列表
order by
    排序字段列表
limit
    分页参数
```
执行顺序：
![[Pasted image 20250727145732.png|100]]

#### LIKE 占位符用于模糊匹配（\_**匹配单个字符，%匹配多个字符)
```xml
<select id="selectUsers" resultType="User">
    SELECT 
        id, name, age, department
    FROM 
        user
    WHERE 
        age > #{minAge}
        <if test="name != null">
            AND name LIKE CONCAT('%',#{name},'%')
        </if>
    GROUP BY 
        department
    HAVING 
        COUNT(*) > 5
    ORDER BY 
        age DESC
    LIMIT 
        #{pageSize} OFFSET #{offset}
</select>
```

#### IN的用法
```Mysql
​**​查指定ID的记录**
SELECT * FROM 员工表 
WHERE 工号 IN ('1001', '1003', '1005')  -- 一次性查这三个员工

**替代多个OR条件**
SELECT * FROM 产品 WHERE 类别 IN ('手机', '平板', '笔记本')
```
```xml
<select id="findUsers" resultType="User">
    SELECT * FROM user
    WHERE id IN  
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}    <!-- MyBatis会自动处理成IN (1,2,3)格式 -->
    </foreach>
</select>
```

#### 聚合函数
| 函数   | 功能     |
|--------|----------|
| count  | 统计数量 |
| max    | 最大值   |
| min    | 最小值   |
| avg    | 平均值   |
| sum    | 求和     |
```mysql
select 聚合函数 from 表名;
select count(*/属性) from 表名
```

#### 分组查询
```mysql
where不可以用聚合函数
having可以用上

select 字段列表 from 表名 [where 条件] group by 分组字段名 [having 分组后过滤条件]

不加gender就不会有前面的属性
select gender, count(*) from emp group by gender
输出：
男   数量
女   数量

-- 根据性别分组，统计男性员工和女性员工的数量
SELECT gender, COUNT(*) FROM emp GROUP BY gender;
-- 根据性别分组，统计男性员工和女性员工的平均年龄
SELECT gender, AVG(age) FROM emp GROUP BY gender;
-- 查询年龄小于45的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址
SELECT workaddress, COUNT(*) AS address_count 
FROM emp 
WHERE age < 45 
GROUP BY workaddress 
HAVING address_count >= 3;
```

#### 分页查询
```mysql
SELECT 字段列表 from 表名 limit 起始索引,查询记录数
```

### 案例练习
```mysql
-- 1. 查询年龄为20,21,22,23岁的女性员工信息
select * from emp where gender = "女",age in ('20', '21', '22','23');

-- 2. 查询性别为男，并且年龄在20-40岁(含)以内的姓名为三个字的员工
select * from emp where gender = '男' AND age BETWEEN 20 AND 40 AND name LIKE '___';

-- 3. 统计员工表中，年龄小于60岁的，男性员工和女性员工的人数
select gender,count(*) from emp where age < 60 group by gender;

-- 4. 查询所有年龄小于等于35岁员工的姓名和年龄，并对查询结果按年龄升序，然后年龄相同，按入职时间降序排序
select name,age from emp where age <= 35 order by age asc,entrydate desc;

-- 5. 查询性别为男，且年龄在20-40岁(含)以内的前5个员工信息，按年龄升序，然后年龄相同，按入职时间降序排序
select * from emp where age BETWEEN 20 and 40 and gender = '男' order by age ASC ,entrydate desc limit 5 
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.EmpMapper">
    <!-- 1. 查询特定年龄女性员工 -->
    <select id="selectFemaleByAges" resultType="Emp">
        SELECT * FROM emp 
        WHERE gender = '女' 
        AND age IN
        <foreach collection="ages" item="age" open="(" separator="," close=")">
            #{age}
        </foreach>
    </select>

    <!-- 2. 复合条件查询男性员工 -->
    <select id="selectMaleByNameLength" resultType="Emp">
        SELECT * FROM emp 
        WHERE gender = '男' 
        AND age BETWEEN #{minAge} AND #{maxAge}
        AND name LIKE '___'
    </select>

    <!-- 3. 统计各性别员工数 -->
    <select id="countByGenderUnder60" resultType="map">
        SELECT gender, COUNT(*) as count 
        FROM emp 
        WHERE age &lt; 60 
        GROUP BY gender
    </select>

    <!-- 4. 查询年轻员工信息（带排序） -->
    <select id="selectYoungEmployees" resultType="Emp">
        SELECT name, age 
        FROM emp 
        WHERE age &lt;= 35
        ORDER BY age ASC, entry_date DESC
    </select>

    <!-- 5. 分页查询男性员工（带排序） -->
    <select id="selectMaleByAgeRange" resultType="Emp">
        SELECT * FROM emp 
        WHERE gender = '男' 
        AND age BETWEEN #{minAge} AND #{maxAge}
        ORDER BY age ASC, entry_date ASC
        LIMIT #{limit}
    </select>

</mapper>
```
