## Centos7服务器硬件安装



### centos7磁盘分区、格式化、挂载

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

