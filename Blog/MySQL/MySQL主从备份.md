### 主服务器备份
```
# mysqldump -uroot -p --all-databases --lock-all-tables > ~/master_db.sql
mysqldump -uroot -p --all-databases > ~/master_db.sql
* --all-databases ：导出所有数据库
* --lock-all-tables ：执行操作时锁住所有表，防止操作时有数据修改
* ~/master_db.sql :导出的备份数据（sql文件）位置，可自己指定

/usr/bin/mysqldump -h'ip' -u'root' -p'' --default-character-set=utf8 db_base --ignore-table=db_base.device_log | gzip > /mnt/ck-data/mysql-backup/backup-data/db_base-${timemy}-bak.sql.gz

/usr/bin/mysqldump -h'ip' -u'root' -p'' --default-character-set=utf8 db_family | gzip > /mnt/ck-data/mysql-backup/backup-data/db_family-${timemy}-bak.sql.gz

# 测试
mysql test -h'ip' -u'ogawadev' -p'123456' -N -e 'SHOW TABLES WHERE tables_in_test NOT LIKE "ss%"' | xargs mysqldump test -h'1ip' -u'uname' -p'123456' >/home/devlop/mysqlfulldump/test.sql
```

### 主服务器配置
```
vim /etc/my.cnf  （MySQL配置文件  ）(vim /usr/my.cnf)

[mysqld]下添加
server-id = 数字                                    （1-254其中之一，如集群内已有的数字不可重复）
log-bin = mysql-bin                                 (定义binlog日志名字=master-bin)
```

#### 重启

```
service mysql restart
```
#### 登录主服务器
```
mysql -u root -p

#创建用于同步的账号,（%代表所有IP，192.168.1.%代表192.168.1.0网段，授权登陆范围自行选择，建议只允许集群内集群）

GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' identified by 'qwer1234';

#主数据库操作（查看二进制日志信息）
show master status;
```

### 从服务器配置
```
vim /etc/my.cnf  （MySQL配置文件  ）(vim /usr/my.cnf)

[mysqld]下添加

server-id = 数字                                    （1-254其中之一，如集群内已有的数字不可重复）

log-bin = mysql-bin                                 (定义binlog日志名字=master-bin)

replicate-wild-ignore-table=db_base.device_log%

replicate-wild-ignore-table=db_base.device_error%
```

#### 重启
```
service mysql restart
```
#### 如果以前执行过配置的话要停止同步

```
stop slave；
```
#### 设置连接到 master 主服务器
```
change master to
master_host='ip',
master_user='slave',
master_password='slave',
master_log_file='master-bin.000001',
master_log_pos=57500;
（主数据库的IP，user=授权的用户，password=授权的密码，log_file=查看到二进制日志文件的名字，log_pos=二进制文件的结束位置）
```

#### 开启同步
```
start slave;
```
#### 查看状态
```
show slave status \G;
```
#### 过滤指定的库和表
```
master

binlog-ignore-db = mysql, information_schema,performance_schema
# 只同步哪些数据库，除此之外，其他不同步
#binlog-do-db = test

  

slave端

#replicate-do-db    设定需要复制的数据库（多数据库使用逗号，隔开）
#replicate-ignore-db 设定需要忽略的复制数据库 （多数据库使用逗号，隔开）
#replicate-do-table  设定需要复制的表
#replicate-ignore-table 设定需要忽略的复制表
#replicate-wild-do-table 同replication-do-table功能一样，但是可以通配符
#replicate-wild-ignore-table 同replication-ignore-table功能一样，但是可以加通配符
#replicate-wild-do-table=db_name.%   只复制哪个库的哪个表
#replicate-wild-ignore-table=mysql.%   忽略哪个库的哪个表
replicate-wild-ignore-table=test.ss%

mysql主从复制不成功_mysql主从同步失败处理
#表示跳过一步错误，后面的数字可变
set global sql_slave_skip_counter =1;
#slave-skip-errors=1062,1053,1146 #跳过指定error no类型的错误
slave-skip-errors=all #跳过所有错误

```

#### 运行中遇到的问题
##### 从数据库遇到的mysql用户没权限
- 第一种
```
the user specified as a definer ('xxx'@'%') does not exist

INSERT INTO `mysql`.`user` (`Host`, `User`, `Password`, `Select_priv`, `Insert_priv`, `Update_priv`, `Delete_priv`, `Create_priv`, `Drop_priv`, `Reload_priv`, `Shutdown_priv`, `Process_priv`, `File_priv`, `Grant_priv`, `References_priv`, `Index_priv`, `Alter_priv`, `Show_db_priv`, `Super_priv`, `Create_tmp_table_priv`, `Lock_tables_priv`, `Execute_priv`, `Repl_slave_priv`, `Repl_client_priv`, `Create_view_priv`, `Show_view_priv`, `Create_routine_priv`, `Alter_routine_priv`, `Create_user_priv`, `Event_priv`, `Trigger_priv`, `Create_tablespace_priv`, `ssl_type`, `ssl_cipher`, `x509_issuer`, `x509_subject`, `max_questions`, `max_updates`, `max_connections`, `max_user_connections`, `plugin`, `authentication_string`, `password_expired`) VALUES ('localhost', 'healthchk', '*714BEF2B582B873F26426F62C1D3C004A22CAAFF', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', 'N', '', '', '', '', '0', '0', '0', '0', 'mysql_native_password', '', 'N');
```

- 第二种
```
Error 'View 'db_family.base_user_third_info' references invalid table(s) or column(s) or function(s) or definer/invoker of view lack rights to use them' on query. Default database: 'db_family'. Query: 'INSERT
```

- 解决办法：安全性改为：invoker