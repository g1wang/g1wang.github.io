# Linux-Primer

## 文件管理
### linux 文件目录结构
/bin bin是Binary的缩写。这个目录存放着最经常使用的命令。

/boot这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。

/dev dev是Device(设备)的缩写。该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。

/etc这个目录用来存放所有的系统管理所需要的配置文件和子目录。

/home用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。

/lib这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。

/lost+found这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。

/media linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。

/mnt系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。

/opt 这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。

/proc这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器：

echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all。

/root该目录为系统管理员，也称作超级权限者的用户主目录。

/sbin s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。

/selinux 这个目录是Redhat/CentOS所特有的目录，Selinux是一个安全机制，类似于windows的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。

/srv 该目录存放一些服务启动之后需要提取的数据。

/sys 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs ，sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统种被创建。

/tmp这个目录是用来存放一些临时文件的。

/usr 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似与windows下的program files目录。

/usr/bin：系统用户使用的应用程序。

/usr/sbin：超级用户使用的比较高级的管理程序和系统守护程序。

/usr/src：内核源代码默认的放置目录。

/var这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。

在linux系统中，有几个目录是比较重要的，平时需要注意不要误删除或者随意更改内部文件。/etc： 上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。/bin, /sbin, /usr/bin, /usr/sbin: 这是系统预设的执行文件的放置目录，比如 ls 就是在/bin/ls 目录下的。值得提出的是，/bin, /usr/bin 是给系统用户使用的指令（除root外的通用户），而/sbin, /usr/sbin 则是给root使用的指令。 /var： 这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在/var/log 目录下，另外mail的预设放置也是在这里。

### ln建立文件连接
Hard Links：上面内容中说过，当系统要读取一个文件时，就会先去读inode table，然后再去根据inode中的信息到块区域去将数据取出来。而hard link 是直接再建立一个inode连接到文件放置的块区域。也就是说，进行hard link的时候实际上该文件内容没有任何变化，只是增加了一个指到这个文件的inode，不过这样一来就会有个问题，因为增加的inode会连接到块区域，而目录本身仅仅消耗inode而已，那么hard link就不能连接目录了。请你记住，hard link 有两个限制：1 不能跨文件系统，因为不通的文件系统有不同的inode table； 2 不能连接目录。

Symbolic Links：跟hard link不同，这个是建立一个独立的文件，而这个文件的作用是当读取这个连接文件时，它会把读取的行为转发到该文件所link的文件上。这样讲，也许比较绕口，那么就来举一个例子。现在有文件a，我们做了一个软连接文件b（只是一个连接文件，非常小），b指向了文件a。当读取b时，那么b就会把读取的动作转发到a上，这样就读取到了文件a。所以，当你删除文件a时，文件b并不会被删除，但是再读取b时，会提示无法打开文件。而，当你删除b时，a是不会有任何影响的。

看样子，似乎 hard link 比较安全，因为即使某一个 inode 被杀掉了，只要有任何一个 inode 存在，那么该文件就不会不见！不过，不幸的是，由于 Hard Link 的限制太多了，包括无法做目录的 link ，所以在用途上面是比较受限的！反而是 Symbolic Link 的使用方向较广！那么如何建立软连接和硬连接呢？这就用到了ln 命令。

#### 软链接
- 更新软链接
`ln -snf  src target`

- 添加软链接
`ln -s src target`

- 添加到系统链接
```
cd /usr/bin
sudo ln -s /usr/local/xx ss
```

## Linux磁盘管理

### 查看磁盘或者目录的容量df 和 du
####  df 
查看已挂载磁盘的总容量、使用容量、剩余容量等，可以不加任何参数，默认是按k为单位显示的。
-h 使用合适的单位显示，例如G，-k -m 分别为使用K，M为单位显示
#### du 
用来查看某个目录所占空间大小，语法：du [-abckmsh]  [文件或者目录名]
- 查看当前目录各文件占用  `sudo du -sh *`

### 磁盘分区、格式化、挂载

#### 分区
- a. 查看磁盘分区表
```
fdisk  -l
```
- b. 查看指定磁盘分区表
```
fdisk  -l  /dev/sdb
```
- c. 分区命令
```
fdisk  /dev/sdb
常用命令：
    n：创建新分区
    d：删除已有分区
    t：修改分区类型
    l：查看所有支持的类型
        p：显示现有分区信息
    w：保存并退出
    q：不保存并退出
    m：查看帮助信息
    
n > 一路默认 > w
```
- d. 使分区生效
```
partprobe  /dev/sdb
```
#### 格式化
- a. 命令
```
mkfs.ext4  /dev/sdb1
```
#### 挂载
- a. 建立一个挂载用的文件夹
```
mkdir  /home/lee/dev
#给分区指定文件系统：ext2和ext3
#mke2fs /dev/sdb1   #默认是ext2，此命令是创建文件系统
#mke2fs -j /dev/sdb1 #-j 是ext3
#mke2fs  -t ext4 /dev/sdb1 #ext4创建文件系统
#tune2fs -l /dev/sdb1  #查看文件系统的详细信息
```
- b. 手动挂载
```
mount  /dev/sdb1  /home/lee/dev
```
- c.开机自动挂载
```
echo  "/dev/sdb  /home/lee/dev  ext4  defaults  0  0"  >>  /etc/fstab
```
- d. 查看挂载的文件夹：
```
ll  /home/lee/dev
```

```
mount  /dev/vdb1  /home/deploy/dataDiskb
```

```
echo  "/dev/vdb1  /home/deploy/dataDiskb  ext4  defaults  0  0"  >>  /etc/fstab
```

### 磁盘扩容

```
1.查看当前的磁盘分区情况
fdisk -l /dev/sda

2.开始进行新的分区
fdisk /dev/sda
 n,p,3,w

3. 重新启动系统
reboot

4. 查看当前的磁盘分区情况
fdisk -l /dev/sda

5. 为这个新分区创建一个物理卷Volume
pvcreate /dev/sda3

6.把物理卷(volume)扩展到新的物理卷，vgdisplay 来查看已有的系统Volume Group的情况
vgdisplay

7.扩展以后的Volume Group到新的物理磁盘卷Volume上
vgextend centos /dev/sda3

8.看看目前已有的逻辑卷(Logic Volume)的情况
lvdisplay

9.扩展逻辑分区
lvextend /dev/centos/root /dev/sda3

10.确认文件系统是xfs
cat /etc/fstab | grep root

11.最后将文件系统resize到新的逻辑卷上来
xfs_growfs /dev/centos/root

12.
reboot

13.
fdisk -l /dev/sda
df -h
```

## VIM编辑器

### 一般模式下查找与替换
```
/word
向光标之后寻找一个字符串名为word的字符串，当找到第一个word后，按”n”继续搜后一个

?word
想光标之前寻找一个字符串名为word的字符串，当找到第一个word后，按”n”继续搜前一个
```

## 文档的压缩与打包
.gz gzip 压缩工具压缩的文件
.bz2 bzip2 压缩工具压缩的文件
.tar tar 打包程序打包的文件(tar并没有压缩功能，只是把一个目录合并成一个文件)
.tar.gz 可以理解为先用tar打包，然后再gzip压缩
.tar.bz2 同上，先用tar打包，然后再bzip2压缩

### gzip
文件压缩后，源文件不存在，慎用。gzip不可以压缩目录
语法：gzip [-d#] filename 其中#为1-9的数字
-d ：解压缩时使用
-# ：压缩等级，1压缩最差，9压缩最好，6为默认
示例：
```
压缩
gzip text1.txt
解压
gzip -d text1.txt.gz
```

### bzip2
语法：bzip2 [-dz] filename
-d ：解压缩
-z ：压缩
```
压缩
bzip2 -z text1.txt
解压
bzip2 -d text1.txt.bz2
```

### tar
语法：tar [-zjxcvfpP] filename
-z ：是否同时用gzip压缩
-j ：是否同时用bzip2压缩
-x ：解包或者解压缩
-t ：查看tar包里面的文件
-c ：建立一个tar包或者压缩文件包
-v ：可视化
-f ：后面跟文件名，压缩时跟-f文件名，意思是压缩后的文件名为filename，解压时跟-f文件名，意思是解压filename。请注意，如果是多个参数组合的情况下带有-f，请把f写到最后面。

示例1: .tar
```
压缩
tar -cvf text.tar text1.txt
解压
tar -xvf text.tar
```

示例2: .tar.gz
```
压缩
tar -zcvf text1.tar.gz text1.txt
解压
tar -zxvf text1.tar.gz text1.txt
```

示例3 tar.bz2
```
压缩
tar -jcvf
解压
tar -jxvf
```

### zip
```
yum install -y unzip zip
```

## 用户管理
### 添加用户
```
useradd devops
#设置密码
passwd devops
```

### 授权
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

## 软件安装

### yum
1. 将旧的yum源移到其他位置，只使用阿里云的yum源安装软件
cd /etc/yum.repos.d
mkdir backup
mv C* backup

2. 下载阿里云的yum源到了/etc/yum.repos.d中，所以不需要切换到/etc/yum.repos.d目录下
CentOS 7
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

## 网络
### scp
- scp 目录
- scp 文件
`scp root@192.168.70.133:/opt/nexusbak.tar.gz /opt/nexus.tar.gz`

### 查看端口占用
`netstat -ano|grep 6379`

### Centos7-防火墙添加端口白名单
CentOS 7默认使用的是firewall作为防火墙 在firewall下开启端口白名单

1.查看下防火墙的状态：
systemctl status firewalld
需要开启防火墙
systemctl start firewalld.service  
查看已开放的端口
firewall-cmd --zone=public --list-ports      

2.添加8484端口到白名单 执行
sudo firewall-cmd --permanent --zone=public --add-port=8484/tcp
提示 success 表示成功
命令含义
--zone # 作用域
--add-port=8484/tcp # 添加端口，格式为：端口/通讯协议
--permanent # 永久生效，没有此参数重启后失效

3.重启防火墙：添加成功之后需要重启防火墙
sudo firewall-cmd --reload

4.其他常用命令：
firewall-cmd --state                          ##查看防火墙状态，是否是running
firewall-cmd --reload                          ##重新载入配置，比如添加规则之后，需要执行此命令
firewall-cmd --get-zones                      ##列出支持的zone
firewall-cmd --get-services                    ##列出支持的服务，在列表中的服务是放行的
firewall-cmd --query-service ftp              ##查看ftp服务是否支持，返回yes或者no
firewall-cmd --add-service=ftp                ##临时开放ftp服务
firewall-cmd --add-service=ftp --permanent    ##永久开放ftp服务
firewall-cmd --remove-service=ftp --permanent  ##永久移除ftp服务
firewall-cmd --add-port=80/tcp --permanent    ##永久添加80端口
firewall-cmd --remove-port=80/tcp --permanent    ##永久添加80端口
firewall-cmd --zone=public --list-ports       ##查看已开放的端口
iptables -L -n                                ##查看规则，这个命令是和iptables的相同的


## 其他
### command not found

```
echo 'export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin' >> /etc/profile
source /etc/profile
```

