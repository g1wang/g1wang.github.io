开始安装
```
# 安装依赖

sudo yum -y install curl policycoreutils policycoreutils-python openssh-server openssh-clients postfix

# 启动ssh

sudo systemctl enable sshd

sudo systemctl start sshd
  
# 将http和https加入防火墙策略，并重启防火墙。

sudo systemctl start firewalld

sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo systemctl reload firewalld

sudo systemctl enable firewalld

# 修改postfix的配置文件

sudo vim /etc/postfix/main.cf

# 使用 /inet_ 进行搜索

inet_interfaces=all

inet_protocols=all

sudo systemctl start postfix

sudo systemctl enable postfix
```

在线安装法
```
# 配置gitlab-ce的yum repo

curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

# 在线安装gitlab-ce

sudo yum install -y gitlab-ce
```
离线安装方法
```
#上清华镜像网址下载gitlab-ce

https://mirrors.tuna.tsinghua.edu.cn/

#将下载下的rpm包放进CentOS中

yum -y localinstall xxx.rpm
```
编写配置文件
```
sudo vim /etc/gitlab/gitlab.rb

# 修改EXTERNAL_URL的值服务器为域名或ip，域名要解析。如果需要外网访问写成localhost或者127.0.0.1。其实这个url是什么不重要，反正外网访问不到，端口才重要！

external_url = 'https://你的ip或解析的域名'

# 重建配置

sudo gitlab-ctl reconfigure
```
Email Settings
```
vim /etc/gitlab/gitlab.rb

#命令行模式输入/Email Settings,修改相关参数。

### Email Settings

gitlab_rails['gitlab_email_enabled'] = true

gitlab_rails['gitlab_email_from'] = 'xxx@qq.com'      # 自己的邮箱

gitlab_rails['gitlab_email_display_name'] = 'Andy'    # gitlab给你发邮件时使用的名字。

# 命令行模式搜索/smtp可快速查找，注意passwd是授权码。（vim小操作：ctrl+v多选，按x删除）

#登录qq邮箱官网，设置---账户---smtp的黄色标签里点击生成授权码，填到smtp_password中  

###! **Use smtp instead of sendmail/postfix.**

gitlab_rails['smtp_enable'] = true

gitlab_rails['smtp_address'] = "smtp.qq.com"

gitlab_rails['smtp_port'] = 465

gitlab_rails['smtp_user_name'] = "xxxx@qq.com"     # 自己的qq邮箱

gitlab_rails['smtp_password'] = "xxxxxxxxxxx"

gitlab_rails['smtp_domain'] = "qq.com"

gitlab_rails['smtp_authentication'] = "login"

gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

# 关闭普罗米修斯（非常吃内存，没有8G内存不要开）

# 在vim的一般模式下输入/Prometheus配合N快速查找
monitor和enable写成false
prometheus['enable'] = false
prometheus['monitor_kubernetes'] = false

# 如果想修改远程仓库的显示地址，修改配置
vim /opt/gitlab/embedded/service/gitlab-rails/config/gitlab.yml.example
 

# 配置完成，保存退出
```
给gitlab添加SSL认证
```
# 修改配置文件：vim /etc/gitlab/gitlab.rb
external_url 'https://10.10.10.63'    #启用https，默认是http
nginx['enable'] = true
nginx['redirect_http_to_https'] = true    #http重定向到https
nginx['ssl_certificate'] = "/etc/gitlab/ssl/server.crt"            #ssl证书路径
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/server.key"    #ssl秘钥路径
  

# 生成ssl证书和私钥

mkdir -p /etc/gitlab/ssl
chmod -R 777 /etc/gitlab/ssl/
openssl req -x509 -nodes -days 10000 -newkey rsa:2048 -keyout /etc/gitlab/ssl/server.key -out /etc/gitlab/ssl/server.crt

安装要求输入国别、城市、机构、姓名等

再次修改权限

chmod -R 777 /etc/gitlab/ssl/
 

参数说明：

openssl：这是用于创建和管理OpenSSL证书，密钥和其他文件的基本命令行工具。

req -x509：这指定我们要使用X.509证书签名请求（CSR）管理。“X.509”是SSL和TLS坚持用于密钥和证书管理的公钥基础结构标准。

-nodes：这告诉OpenSSL跳过用密码保护我们的证书的选项。当服务器启动时，我们需要Apache能够读取文件，而无需用户干预。密码可以防止这种情况发生，因为每次重新启动后我们都必须输入密码。

第365天：此选项设置证书被视为有效的时间长度。我们在这里定了一年。

-newkey rsa：2048：这指定我们要同时生成一个新的证书和一个新的密钥。我们没有在上一步创建签名证书所需的密钥，所以我们需要与证书一起创建证书。该rsa:2048部分告诉它做一个2048位长的RSA密钥。

-keyout：这一行告诉OpenSSL在哪里放置我们正在创建的私有密钥文件。

-out：这告诉OpenSSL在哪里放置我们正在创建的证书。
```
修改好配置文件后
```
# 重建配置
gitlab-ctl reconfigure
```
登录gitlab
```
https://ip 或者 https://域名 如果浏览器报不安全，忽略，继续进入。
案例：我搭建成功的网址 https://www.8zi.site
```
给gitlab添加SSH key
```
# 在某个开发人员的机器上：
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub
# 将id_rsa.pub文件内容全部复制
# 浏览器登录gitlab，进入gitlab
# Profile Settings-->SSH Keys--->Add SSH Key

#粘贴你复制的公钥，点击add按钮。
```
Gitlab常用命令
```
gitlab-ctl start         # 启动所有 gitlab 组件
gitlab-ctl stop          # 停止所有 gitlab 组件
gitlab-ctl restart       # 重启所有 gitlab 组件
gitlab-ctl status        # 查看服务状态
gitlab-ctl reconfigure   # 启动服务
gitlab-ctl show-config   # 验证配置文件
gitlab-ctl tail          # 查看日志 
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab
vim /etc/gitlab/gitlab.rb # 修改默认的配置文件
```















