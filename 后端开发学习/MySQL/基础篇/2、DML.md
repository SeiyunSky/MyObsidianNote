用来对数据库中表的数据记录进行增删改操作

### 添加数据
![[Pasted image 20250727142759.png]]
```xml
一般情况
<insert id="insertUser" parameterType="com.example.User"> INSERT INTO user (name, age, email) VALUES (#{name}, #{age}, #{email}) </insert>

返回自增主键
<insert id="insertUser" useGeneratedKeys="true"keyProperty="id">
    INSERT INTO user (...) VALUES (...)
</insert>

批量操作
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO user (name, age) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.name}, #{user.age})
    </foreach>
</insert>
```
### 修改数据
```Mysql
UPDATE 表名 SET 字段名1=值1,字段名2=值2,···[Where 条件];
```
```xml
一般情况
<update id="updateUser" parameterType="com.example.User">
    UPDATE user SET
    name = #{name},
    age = #{age},
    WHERE id = #{id}
</update>

增加判断语句
<update id="updateUserSelective" parameterType="User">
    UPDATE user
    <set>
        <if test="name != null">name = #{name},</if>
        <if test="age != null">age = #{age},</if>
    </set>
    WHERE id = #{id}
</update>
```

### 删除数据
```mysql
DELETE FROM 表名 [Where 条件]
```
```xml
基础删除
<delete id="deleteById">
    DELETE FROM user 
    WHERE id = #{id}
</delete>

 动态条件删除
<delete id="deleteByCondition" parameterType="map">
    DELETE FROM user
    <where>
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age > #{age}</if>
    </where>
</delete>

批量删除 (IN语句)
<delete id="batchDelete">
    DELETE FROM user 
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```
