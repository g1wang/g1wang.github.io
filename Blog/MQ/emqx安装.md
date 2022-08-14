## emqx安装
### 安装 openssl-1.1.1g

```
rpm -aq|grep openssl
openssl-1.0.1e-58.el6_10.x86_64
openssl-devel-1.0.1e-58.el6_10.x86_64

wget https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar -xvf openssl-1.1.1g.tar.gz
cd openssl-1.1.1g
./config shared --openssldir=/usr/local/openssl --prefix=/usr/local/openssl
sudo make
sudo make install
echo "/usr/local/openssl/lib/" >> /etc/ld.so.conf
ldconfig
```

### 安装 emqx 

[官网文档](https://www.emqx.com/zh/downloads-and-install?product=broker&version=4.4.1&os=Centos7&oslabel=CentOS%207)

```
wget https://www.emqx.com/zh/downloads/broker/4.4.1/emqx-4.4.1-otp24.1.5-3-el7-amd64.rpm
sudo yum install emqx-4.4.1-otp24.1.5-3-el7-amd64.rpm
sudo emqx start
```



