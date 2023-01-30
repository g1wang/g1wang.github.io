## jenkins 安装
一 更换yum源
1. 将旧的yum源移到其他位置，只使用阿里云的yum源安装软件
cd /etc/yum.repos.d
sudo mkdir backup
sudo mv C* backup
 
2. 下载阿里云的yum源到了/etc/yum.repos.d中，所以不需要切换到/etc/yum.repos.d目录下
CentOS 7
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

二 安装 jenkins 

// 如果未安装java还需安装java
sudo yum install java-11-openjdk* -y
java -version
// 需要切换使用jdk11
sudo alternatives --config java
// 选择jdk11


sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo --no-check-certificate
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key

//错误：依赖检测失败：daemonize 被 jenkins-2.303.1-1.1.noarch 需要
sudo yum -y install epel-release
sudo yum -y install daemonize
sudo yum install jenkins


三 相关命令
// 启动和停止
sudo service jenkins start
sudo service jenkins stop
sudo service jenkins restart

sudo chkconfig jenkins on

//启动失败，可以运行如下命令查看错误信息
sudo systemctl status jenkins.service

//需要修改jenkins配置文件
sudo vim /etc/rc.d/init.d/jenkins
//添加防火墙端口参考  linux防火墙
sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
sudo firewall-cmd --reload

四、启动jenkins
//ip:port
192.168.31.67:8080

五、安装插件
1 安装和Git，GitLab插件
//安装git客户端
sudo yum install git

2、安装git插件
系统管理=》管理插件 git gitlab


3 配置gitLab连接
系统管理=》系统配置
        name：可以随便写一个
　　 host URL：host地址注意只填写host不要库地址写http地址即可
　　 credentials：选择一个证书
3.1 证书
        kind：选择GitLab API token
         API token：输入token  （gitlab Access Tokens）//需要记录生成的token xNee6mzoJJdHJewESTza

六 新建item

1、生成ssh key
>  vi /etc/passwd 查看一下jenkins账号的home目录
>ssh-keygen -t rsa -C "root@192.168.3167" -b 4096 # 生成公钥私钥，注意双引号内是个助记符根据需要修改
>cd /root/.ssh # 进入ssh目录
>git ls-remote -h git@192.168.31.172:wanggl/demo.git # 连一下git服务器，生成known_hosts文件
>ll

2 进入jenkins目录将刚才创建的sshkey复制过来并将所有者指到jenkins账号
> cd /var/lib/jenkins # 进入jenkins的home目录
> mkdir .ssh # 创建ssh目录存放sshkey文件，如果存在会报错
> cd .ssh
> cp /root/.ssh/* . # 将root账号下的sshkey文件复制过来，此时如果执行ll看一下这两个文件所有这应该是root
> chgrp jenkins * # 将key文件的组改为jenkins
> chown jenkins * # 将key文件的所有者改为jenkins


3 打开 id_rsa.pub 将其中内容复制到记事本中，然后再copy到git服务器上,访问gitlab将刚才生成的公钥添加到ssh keys中

4 创建证书
global -> Add credentials新建一个证书
　            kind：选择 SSH Username with private key
　　　　Username：随便输入，之后在创建item是记得住选择那个即可
　　　　Priveate key：选择“Enter directly”后输入私钥，记住一定是私钥 （按照上步骤在linux上生成密钥后，会是一对其中带pub后缀的是公钥。id_rsa私钥、id_rsa.pub公钥)。在linux执行cat id_rsa将所有内容copy填入key中
　　　　Passphrase：不填，如果填了每次都需要输入密码
　　　　填好后点击“ok”保存

5 新建item
输入任务名称，选择“构建一个自由风格软件项目”后单击“确定”
在GitLab connection处选择刚才创建的连接（输入git host时创建的连接）
　　　　选择git并Credentials处选择刚才输入私钥的证书
　　　　Repository URL输入git项目地址（注意输入时需添加ssh注意如果修改端口应填写git@192.168.31.172:wanggl/demo.git）
　　　　之后单击“保存”即可。 

6  构建job 
点击“立即构建”
　　　　由于item只配置了git所以只会clone git，clone后在如下目录中，以job名称为目录保存
　　　　 /var/lib/jenkins/workspace





