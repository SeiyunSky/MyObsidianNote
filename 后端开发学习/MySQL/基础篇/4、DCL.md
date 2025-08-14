DataControl Language 用来管理数据库用户

```MYSQL
1. 查询用户
USE mysql;
SELECT * FROM user;

2. 创建用户
CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';

3. 修改用户密码
ALTER USER '用户名'@'主机名' IDENTIFIED WITH mysql_native_password BY '新密码';

4. 删除用户
DROP USER '用户名'@'主机名';
```