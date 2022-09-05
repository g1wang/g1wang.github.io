1. 将旧的yum源移到其他位置，只使用阿里云的yum源安装软件

cd /etc/yum.repos.d

mkdir backup

mv C* backup

2. 下载阿里云的yum源到了/etc/yum.repos.d中，所以不需要切换到/etc/yum.repos.d目录下

CentOS 7

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
