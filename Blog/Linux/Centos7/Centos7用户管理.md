### Centos7用户管理

#### 添加用户

`useradd devops`

#### 授权

```
#添加 sudoers 文件可写权限
chmod -v u+w /etc/sudoers
#修改 sudoers 文件
vi /etc/sudoers
#添加授权
#[用户名]    ALL=(ALL)    ALL 
devops    ALL=(ALL)       ALL
#收回 sudoers 文件可写权限
chmod -v u-w /etc/sudoers
```



