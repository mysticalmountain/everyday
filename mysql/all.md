# 创建数据库、用户、授权
```sql
create database dbxxx;
create user uxxx identified by 'xxx';
grant all privileges on dbxxx.* to 'uxxx'@'%' identified by 'xxx';
flush privileges;
```

# 删除用户
```sql
use mysql
delete from user wehre User='uxxx';
flush privileges;
```

# 链接mysql
```sql
mysql -u root -h 127.0.0.1 -p
```

# 设置数据库编码
修改my.cnf
```sql
vi /etc/my.cnf
在[client]下添加
default-character-set=utf8
在[mysqld]下添加
default-character-set=utf8

修改表编码
alter table score default character set utf8;  
查看表编码
show create table score; 
修改列编码
alter table score change score score varchar(50) character utf8;  
```

