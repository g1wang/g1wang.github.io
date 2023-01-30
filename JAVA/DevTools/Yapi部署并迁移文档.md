  

# Yapi部署并迁移文档

### 安装node.js 8.15

  


```
#最新版

https://nodejs.org/dist/v10.15.0/node-v10.15.0-darwin-x64.tar.gz

#8.15版本

https://nodejs.org/dist/latest-v8.x/node-v8.15.0-linux-x64.tar.gz

#解压8.15版本

tar xvf node-v8.15.0-linux-x64.tar.gz

mv node-v8.15.0-linux-x64 /etc/nodejs

#编辑环境变量

vim /etc/profile

export PATH=$PATH:/etc/nodejs/bin

#使生效

source /etc/profile

node -v
```

  

### 安装mongodb4.05

  

```


#安装最新版mongodb

#下载速度慢，可直接拷贝当前yapi的/home/wwwroot/xxx/server/mongodb文件夹

wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.0.5.tgz

#解压mongodb

tar xvf mongodb-linux-x86_64-rhel70-4.0.5.tgz

mv mongodb-linux-x86_64-rhel70-4.0.5 /home/wwwroot/xxx/server/mongodb

#创建mongodb 日志和数据文件夹

mkdir -p /home/wwwroot/xxx/logs/mongo

home/wwwroot/xxx/database/mongodb

#编写mongodb配置文件

vim /home/wwwroot/xxx/server/mongodb/bin/mongod.conf

port=27017

bind_ip=127.0.0.1

logpath=/home/wwwroot/xxx/logs/mongo/mongod.log

logappend=true

fork=true

dbpath=/home/wwwroot/xxx/database/mongodb

  

#运行mongodb

./mongod -f mongod.conf

#创建数据库以及数据库用户

./mongo

use admin

use yapi

db.createUser({'user':'root','pwd':'root','roles':[{ "role" : "readWrite", "db" : "yapi" }]})

  

#源服务器中导出源yapi平台数据

mongodump -d yapi -o /home/yapi_data

#将数据传输到新yapi服务器中

#新服务器中导入源yapi平台数据

mongorestore -d yapi --drop /home/yapi_data

```
  

  

### 安装git和pm2

  
```
yum install -y git

npm install -g pm2 --registry=https://registry.npm.taobao.org
```



  

### 安装并启动yapi

```  

#直接复制源服务器/home/wwwroot/xxx/server/yapi文件夹

mv yapi /home/wwwroot/xxx/server/yapi

#启动yapi

/home/wwwroot/xxx/server/yapi/vendors/server

pm2 start app.js
```

  

### 配置nignx

```
upstream yapi{

server 127.0.0.1:3000;

}

server {

listen 80;

server_name localhost;

location / {

proxy_pass http://yapi;

}

#重新加载nginx 配置

nginx -s reload

#访问yapi

http://ip:80  
```



