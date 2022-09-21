# MongoDB-CentOS7-安装

## 安装 mongodb
### 下载
-   官网地址：[https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)

-   下载：[https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.5.tgz](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-5.0.5.tgz)

```
sudo tar -zxvf mongodb-linux-xxx.tgz
sudo mkdir /usr/local/mongodb
mv mongodb-linux-x86_64-3.6.13/* /usr/local/mongodb
export PATH=/usr/local/mongodb/bin:$PATH 
cd /usr/local/mongodb

#创建data/db、data/logs文件夹用来存放数据和日志
sudo mkdir -p data/db
sudo mkdir -p data/logs

# 设置权限
sudo chown devops /usr/local/mongodb/data/db
sudo chown devops /usr/local/mongodb/data/logs

# 配置文件
# sudo vim /usr/local/mongodb/mongodb.conf
dbpath=/usr/local/mongodb/data/db
logpath=/usr/local/mongodb/data/logs/mongodb.log
port=27017
auth=true
fork=true
bind_ip=0.0.0.0

# 启动
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/mongodb.conf

# 添加管理员
# (mongoDB 没有无敌用户root，只有能管理用户的用户 userAdminAnyDatabase)
./mongo
use admin    

db.createUser( {user: "root",pwd: "root",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]});

use yapi    
db.createUser( {user: "yapi",pwd: "yapi",roles: [ { role: "readWrite", db: "yapi" } ]});
mongodump -h 127.0.0.1 -u yapi -p yapi -d yapi -o /home/yapi.bak
mongorestore -h 127.0.0.1:27107  -u yapi -p yapi -d yapi /home/devops/Documents/yapi.bak/  

# 配置环境变量
sudo vim /etc/profile
#MONGO_HOME path
export MONGO_HOME=/usr/local/mongodb
export PATH=$PATH:$MONGO_HOME/bin
source /etc/profile

# 将mongo路径软链到/usr/bin路径下，方便随处执行mongo命令
sudo ln -s /usr/local/mongodb/bin/mongo  /usr/bin/mongo


# 开机启动

sudo vim /etc/rc.d/init.d/mongod
start() {  
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/mongodb.conf
}  
stop() {  
/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/mongodb.conf  --shutdown  
}  
case "$1" in  
  start)  
start  
;;  
stop)  
stop  
;;  
restart)  
stop  
start  
;;  
  *)  
echo  
$"Usage: $0 {start|stop|restart}"  
exit 1  
esac
  
# 权限chmod
sudo chmod +x /etc/rc.d/init.d/mongod

# 验证
lsof -i :27017
```

## MongoDB Database Tools 安装
- 官网：[https://www.mongodb.com/try/download/database-tools](https://www.mongodb.com/try/download/database-tools)
 下载后解压到mongodb/bin 下面
```
sudo wget https://fastdl.mongodb.org/tools/db/mongodb-database-tools-rhel72-s390x-100.5.1.tgz
sudo tar -zxvf mongodb-database-tools-rhel72-s390x-100.5.1.tgz
sudo cp ./* /usr/local/mongodb/bin/
```