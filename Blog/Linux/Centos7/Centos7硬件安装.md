## Centos7磁盘管理

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

### centos7磁盘扩容

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