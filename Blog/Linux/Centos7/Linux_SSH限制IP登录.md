### Linux_SSH限制IP登录

#### 1、限制用户SSH登录

只允许指定用户进行登录（白名单）：
在/etc/ssh/sshd_config配置文件中设置AllowUsers选项，（配置完成需要重启 SSHD 服务）格式如下：
```
AllowUsers aliyun test@192.168.1.1
# 允许 aliyun 和从 192.168.1.1 登录的 test 帐户通过 SSH 登录系统。
```

只拒绝指定用户进行登录（黑名单）：
在/etc/ssh/sshd_config配置文件中设置DenyUsers选项，（配置完成需要重启SSHD服务）格式如下：
```
DenyUsers zhangsan aliyun #Linux系统账户
# 拒绝 zhangsan、aliyun 帐户通过 SSH 登录系统
```

重启SSH
```
service sshd restart
```

#### 2、限制IP SSH登录
说明：这里的IP是指客户端IP，不是服务器IP，下面的例子使用了hosts.allow文件的配置方式，目的是快，但也有不灵活的，建议改成iptables的方案。

除了可以禁止某个用户登录，我们还可以针对固定的IP进行禁止登录，linux 服务器通过设置/etc/hosts.allow和/etc/hosts.deny这个两个文件，hosts.allow许可大于hosts.deny可以限制或者允许某个或者某段IP地址远程 SSH 登录服务器，方法比较简单，且设置后立即生效，不需要重启SSHD服务，具体如下：

/etc/hosts.allow添加
```
sshd:192.168.0.1:allow
#允许 192.168.0.1 这个IP地址SSH登录

sshd:192.168.0.:allow
#允许192.168.0.1/24这段IP地址的用户登录，多个网段可以以逗号隔开，比如#192.168.0.,192.168.1.:allow
```

hosts.allow和hosts.deny两个文件同时设置规则的时候，hosts.allow文件中的规则优先级高，按照此方法设置后服务器只允许192.168.0.1这个IP地址的SSH登录，其它的IP都会拒绝。
/etc/hosts.deny添加
```
sshd:ALL #拒绝全部IP
```