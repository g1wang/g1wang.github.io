## Tengine安装 [centos 7]

### 说明
官网：http://tengine.taobao.org/
版本：Tengine-2.3.3

### 安装
#### 1.下载
```
cd  /usr/local/src/
wget http://tengine.taobao.org/download/tengine-2.3.3.tar.gz
tar zxvf Tengine-2.3.3.tar.gz
```
#### 2.配置检查 
```
# 缺少的包
yum -y install gcc
yum -y install pcre pcre-devel
yum -y install openssl openssl-devel

# 配置检查
cd Tengine-2.3.3
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-syslog--with-http_lua_module
--with-http_stub_status_module
--with-syslog--with-http_lua_module
```
#### 3.编译安装
```
sudo make
sudo  make install
```

#### 4.添加自启动服务

##### 添加 /etc/init.d/nginx
```
vim /etc/init.d/nginx
```
##### 修改内容
```
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
```

##### 最终版本

```
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:.*--user=" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -n "$user" ]; then
      if [ -z "`grep $user /etc/passwd`" ]; then
         useradd -M -s /bin/nologin $user
      fi
      options=`$nginx -V 2>&1 | grep 'configure arguments:'`
      for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
              value=`echo $opt | cut -d "=" -f 2`
              if [ ! -d "$value" ]; then
                  # echo "creating" $value
                  mkdir -p $value && chown -R $user $value
              fi
          fi
       done
    fi
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $prog -HUP
    retval=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
##### 修改conf/nginx.conf

```
将  #pid=logs/nginx.pid（可以看到pid默认生成在logs目录下） 修改为 pid=/var/run/nginx.pid 即可 去掉注释的#   重启即可
```

##### 设置开机启动

```
chkconfig --add nginx
chkconfig nginx on
```

#### 5.nginx命令参数
```
nginx -m 显示所有加载的模块
nginx -l 显示所有可以使用的指令
nginx -t 检查nginx的配置文件是否正确
nginx  启动nginx
nginx -s reload 重启nginx
nginx -s stop 停止nginx
```