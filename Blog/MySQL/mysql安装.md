## mysql5.6 安装



### 1、卸载系统自带的MariaDB。

- （1）查看有没有安装mariadb
```
rpm -qa|grep -i mariadb
```

- （2）卸载mariadb：

```
sudo rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```

- （3）删除my.conf：

```
sudo rm /etc/my.cnf
```

### 2、卸载已安装的mysql（没安装过忽略）

- （1）查看有没有安装mysql：`rpm -qa|grep -i mysql`
- （2）如果有，卸载：`rpm -e MySQL-client-5.6.38-1.el7.x86_64`
- （3）查看mysql服务：`chkconfig --list | grep -i mysql`
- （4）删除mysql服务：`chkconfig --del mysql`
- （5）查找mysql分散的文件：`whereis mysql   （或者用命令 find / -name *mysql*）`
- （6）根据查询到的文件目录删除：`rm -rf /usr/lib/mysql`

### 3、安装mysql 

- （1）官网下载安装包MySQL-5.6.44-1.el7.x86_64.rpm-bundle.tar，

  - 下载地址为：https://dev.mysql.com/downloads/mysql
```
cd /usr/local/src 

sudo wget https://cdn.mysql.com/archives/mysql-5.6/MySQL-5.6.51-1.el7.x86_64.rpm-bundle.tar

```

- （2）创建用户组：
```
sudo groupadd mysql
```

- （3）创建一个用户名为mysql的用户并加入mysql用户组：
```
sudo useradd -g mysql mysql
```

- （4）安装依赖： 
```
sudo yum install perl  yum -y install autoconf
```

- （5）并进入该目录解压：
```
sudo tar -xvf MySQL-*-bundle.tar*
```

- （6）运行命令进行安装：

```
sudo rpm -ivh MySQL-client-*.el7.x86_64.rpm
sudo rpm -ivh MySQL-devel*1.el7.x86_64.rpm
sudo rpm -ivh MySQL-server-*.el7.x86_64.rpm
```

  

### 4、修改密码。

- `root默认密码 sudo cat /root/.mysql_secret`

（1）查看mysql服务状态：

```
systemctl status mysql
mysql -u root -p
```

（2）执行以下命令修改密码：

`	SET PASSWORD = PASSWORD('123456');`

（3）Mysql添加用户，给用户授权

```
#创建用户
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
CREATE USER 'javacui'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'javacui'@'172.20.0.0/255.255.0.0' IDENDIFIED BY '123456';
CREATE USER 'javacui'@'%' IDENTIFIED BY '123456';
CREATE USER 'javacui'@'%' IDENTIFIED BY '';CREATE USER 'javacui'@'%';
\#授权
GRANT privileges ON databasename.tablename TO 'username'@'host';
\#刷新权限flush privileges;
\#设置与更改用户密码
SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
```

### 5、设置mysql开机启动

- 查看服务：

  `chkconfig --list mysql`

- 开启mysql服务自动启动：

  `chkconfig mysql on`

### 6 防火墙开发端口
- [Centos7 防火墙添加端口白名单](https://app.yinxiang.com/shard/s33/nl/9786851/4e2fb976-00d4-4195-bb00-e524439db291)

### 7、mysql命令及配置：

数据库文件：

配置文件：默认配置文件 /usr/my.cnf