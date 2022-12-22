# Redis-install

## 安装依赖

### 安装gcc

```
# 检查是否安装gcc
gcc -v
# 未安装，执行安装
yum install -y gcc
```

## 安装Redis
```
# 下载，选择redis7
https://redis.io/download/
# 上传redis-7.0.7.tar.gz到服务器
cd /home/devops/middleware/
# 解压
tar -zxvf redis-7.0.7.tar.gz
# 执行编译
cd redis-7.0.7
make
# 安装并指定安装目录
make install PREFIX=/usr/local/redis
```

## 启动Redis

### 1.前台启动
```
cd /usr/local/redis/bin/
```

### 2.后台启动
```
cp /home/devops/middleware/redis-7.0.7/redis.conf /usr/local/redis/bin/
# 修改 redis.conf 文件，把 daemonize no 改为 daemonize yes
vim redis.conf
# 启动
./redis-server redis.conf
```

## 设置开机启动

### 添加开机启动服务
```
vim /etc/systemd/system/redis.service
```

添加内容
```
[Unit]
Description=redis-server
After=network.target
 
[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/bin/redis.conf
PrivateTmp=true
 
[Install]
WantedBy=multi-user.target
```

设置开机启动
```
# 开机自动启动
systemctl daemon-reload
# 启动redis服务
systemctl start redis.service
# 查看服务状态
systemctl enable redis.service
# 停止服务 
systemctl stop redis.service
# 取消开机自动启动(卸载服务) 
systemctl disabled redis.service
```