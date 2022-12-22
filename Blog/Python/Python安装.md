# Python安装

## 安装依赖
```
sudo yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make
```

## 安装epel扩展源，相当于新增加了一个yum源
```
sudo yum -y install epel-release
```

## 安装pip
```
sudo yum -y install python-pip
```

## 安装python
```
# 下载
mkdir /home/devops/middleware/python
cd /home/devops/middleware/python
wget https://www.python.org/ftp/python/3.11.1/Python-3.11.1.tar.xz

# 解压
tar  -Jcvf Python-3.11.1.tar.xz

cd Python-3.11.1
# 切换到 root
su
# 编译安装
./configure prefix=/usr/local/python3
make && make install
```

## 软链接python3复制到/usr/bin/目录
```
cd /usr/local/python3/bin/ cp python3 /usr/bin/
cp python3 /usr/bin/
```
## 验证是否安装成功
```
Python3 -V

# 结果
Python 3.11.1
```