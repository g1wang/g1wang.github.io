# MySQL工具

## mysql运维工具 percona-toolkit
下载地址： https://www.percona.com/downloads/percona-toolkit/LATEST/
安装依赖
```
yum install perl-IO-Socket-SSL perl-DBD-MySQL perl-Time-HiRes perl perl-DBI perl-Digest-MD5 -y
# 下载安装包,解压,进入bin目录直接使用
```

### pt-online-schema-change
功能可以在线整理表结构，收集碎片，给大表添加字段和索引。避免出现锁表导致阻塞读写的操作。针对 MySQL 5.7 版本，就可以不需要使用这个命令，直接在线 online DDL 就可以了

```
# 增加字段
./pt-online-schema-change --user=devlop --password=qwer1234 --host=192.168.86.90 --port=3306 --alter="ADD COLUMN city_id INT" D=recsys,t=user_info --execute
```


### pt-query-digest
功能：现在捕获线上TOP 10 慢 sql 语句
```
./pt-query-digest --since=24h /data/mysql/slow.log > 1.log
```