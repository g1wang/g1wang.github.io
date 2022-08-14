## 服务器环境安装

### 一、JDK安装配置

#### 1、安装包

```
路径：/usr/local/java
解压：tar ‐zxvf jdk-8u341-linux-x64.tar.gz
解压后：/usr/local/java/jdk1.8.0_341
```

#### 2、环境配置 /etc/profile

```
vim /etc/profile

#追加内容
JAVA_HOME=/usr/local/java/jdk1.8.0_341 
JRE_HOME=/usr/local/java/jdk1.8.0_341/jre 
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH

#生效
source /etc/profile
```

