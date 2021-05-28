### 查看数据库时区

```
mysql> show variables like '%time_zone%';
 +------------------+--------+
 | Variable_name | Value | 
 +------------------+--------+ 
 | system_time_zone | CST |
 | time_zone | CST -07:00|
 +------------------+--------+ 
 2 rows in set (0.00 sec)   这里CST -07:00代表北美时间
 ```


### 通过sql命令临时修改

```
# 设置全局时区 mysql> set global time_zone = '+8:00';
Query OK, 0 rows affected (0.00 sec) 
# 设置时区为东八区 mysql> set time_zone = '+8:00'; 
Query OK, 0 rows affected (0.00 sec) 
# 刷新权限使设置立即生效 mysql> flush privileges; 
Query OK, 0 rows affected (0.00 sec)
mysql> show variables like '%time_zone%';
 +------------------+--------+
 | Variable_name | Value |
 +------------------+--------+
 | system_time_zone | EST |
 | time_zone | +08:00 | 
 +------------------+--------+
 2 rows in set (0.00 sec)
```

### 修改my.cnf实现永久修改
```
vim /etc/mysql/my.cnf
```
然后在mysqld下边的配置中添加一行
```
default-time_zone = '+8:00'
```
重启mysql
```
service mysql restart
```