## DockerPrimer

### CentOS 安装 Docker

#### 系统要求

Docker 支持 64 位版本 CentOS 7/8，并且要求内核版本不低于 3.10。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 `overlay2` 存储层驱动）无法使用，并且部分功能可能不太稳定

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

#### 安装方式1：使用 yum 安装

```
sudo yum install -y yum-utils
```

鉴于国内网络问题，强烈建议使用国内源

```
sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

sudo sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

# 官方源
# sudo yum-config-manager \
#     --add-repo \
#     https://download.docker.com/linux/centos/docker-ce.repo

```

更新 `yum` 软件源缓存，并安装 `docker-ce`

```
yum clean all
yum makecache
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

#### 安装方式2：使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装，另外可以通过 `--mirror` 选项使用国内源进行安装

```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```


#### 启动docker
```
sudo systemctl enable docker
sudo systemctl start docker
```

#### 建立 `docker` 组
默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组
```
sudo groupadd docker
sudo usermod -aG docker $USER
```

#### 检测是否安装成功
```
docker run --rm hello-world
```

#### 镜像加速器

1.查看是否在 `docker.service` 文件中配置过镜像地址

```
systemctl cat docker | grep '\-\-registry\-mirror'
```

如果该命令有输出，那么请执行 `$ systemctl cat docker` 查看 `ExecStart=` 出现的位置，修改对应的文件内容去掉 `--registry-mirror` 参数及其值

2.修改 /etc/docker/daemon.json

```
sudo vim /etc/docker/daemon.json

#添加
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```

3.重启服务

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

4.其他镜像服务

```
腾讯云：
{
  "registry-mirrors": [
    "https://mirror.ccs.tencentyun.com“
  ]
}
```

### 镜像
#### 获取镜像
##### 拉去镜像
```
# docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
docker pull ubuntu:18.04

```
##### 运行镜像
```
docker run ubuntu:18.04
```

#### 使用 Dockerfile 定制镜像


### 操作容器
#### 启动 
```
# docker run
docker run ubuntu:18.04

# -d 后台运行
docker run -d ubuntu:18.04
```
当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

-   检查本地是否存在指定的镜像，不存在就从 [registry](/docker_practice/repository) 下载
-   利用镜像创建并启动一个容器
-   分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
-   从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
-   从地址池配置一个 ip 地址给容器
-   执行用户指定的应用程序
-   执行完毕后容器被终止

#### 终止
```
# docker container stop
# docker container start
# docker container restart

```

### 数据管理
#### 数据卷
`数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：
-   `数据卷` 可以在容器之间共享和重用
-   对 `数据卷` 的修改会立马生效
-   对 `数据卷` 的更新，不会影响镜像
-   `数据卷` 默认会一直存在，即使容器被删除

> 注意：`数据卷` 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）。

##### 创建一个数据卷
```
#创建
sudo docker volume create my-vol

#查看所有的数据卷
sudo docker volume ls

#查看指定 `数据卷` 的信息
sudo docker volume inspect my-vol
```

##### 启动一个挂载数据卷的容器
在用 `docker run` 命令的时候，使用 `--mount` 标记来将 `数据卷` 挂载到容器里。在一次 `docker run` 中可以挂载多个 `数据卷`。
下面创建一个名为 `web` 的容器，并加载一个 `数据卷` 到容器的 `/usr/share/nginx/html` 目录
```
sudo docker run -d -P \
    --name web \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

##### 查看数据卷的具体信息
```
docker inspect web
```

##### 删除数据卷
```
# 删除指定的数据卷
docker volume rm my-vol

# 清理无主的数据卷
sudo docker volume prune
```

#### 挂载主机目录
