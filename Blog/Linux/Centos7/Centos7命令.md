# Centos7命令

## command not found...

```
echo 'export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin' >> /etc/profile
source /etc/profile
```



## 查看端口占用

`netstat -ano|grep 6379`

##  软链接

### 更新软链接
`ln -snf  src target`

### 添加软链接
`ln -s src target`

### 添加到系统链接
```
cd /usr/bin
sudo ln -s /usr/local/xx ss
```

## 压缩

### tar

```
tar  -zcvf  文件名.tar.gz 输入被压缩的文件名，可以多个
```

### zip
```
yum install -y unzip zip
```


## scp

- scp 目录
- scp 文件
scp root@192.168.70.133:/opt/nexusbak.tar.gz /opt/nexus.tar.gz