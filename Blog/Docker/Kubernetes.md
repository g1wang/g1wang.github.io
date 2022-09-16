# Kubernetes
开源容器编排引擎

## 简介
管理跨多个主机的容器，提供基本的部署，维护以及应用伸缩
-   便携：支持公有云，私有云，混合云，以及多种云平台
-   可拓展：模块化，可插拔，支持钩子，可任意组合
-   自修复：自动重调度，自动重启，自动复制

## 基本概念

![](./image/Kubernetes1-1.png)

-  节点（`Node`）：一个节点是一个运行 Kubernetes 中的主机。
-  容器组（`Pod`）：一个 Pod 对应于由若干容器组成的一个容器组，同个组内的容器共享一个存储卷(volume)。
-  容器组生命周期（`pos-states`）：包含所有容器状态集合，包括容器组状态类型，容器组生命周期，事件，重启策略，以及 replication controllers。
-  Replication Controllers：主要负责指定数量的 pod 在同一时间一起运行。
-  服务（`services`）：一个 Kubernetes 服务是容器组逻辑的高级抽象，同时也对外提供访问容器组的策略。
-  卷（`volumes`）：一个卷就是一个目录，容器对其有访问权限。
-  标签（`labels`）：标签是用来连接一组对象的，比如容器组。标签可以被用来组织和选择子对象。
-  接口权限（`accessing_the_api`）：端口，IP 地址和代理的防火墙规则。
-  web 界面（`ux`）：用户可以通过 web 界面操作 Kubernetes。
-  命令行操作（`cli`）：`kubectl`命令。

### 节点
在 `Kubernetes` 中，节点是实际工作的点，节点可以是虚拟机或者物理机器，依赖于一个集群环境。每个节点都有一些必要的服务以运行容器组，并且它们都可以通过主节点来管理。必要服务包括 Docker，kubelet 和代理服务。

#### 容器状态
容器状态用来描述节点的当前状态。现在，其中包含三个信息：
  - 主机IP: 主机 IP 需要云平台来查询，`Kubernetes` 把它作为状态的一部分来保存。如果 `Kubernetes` 没有运行在云平台上，节点 ID 就是必需的
  - 节点周期: 通常来说节点有 `Pending`，`Running`，`Terminated` 三个周期
  - 节点状态: 节点的状态主要是用来描述处于 `Running` 的节点。当前可用的有 `NodeReachable` 和 `NodeReady`

####节点管理
节点并非 Kubernetes 创建，而是由云平台创建，或者就是物理机器、虚拟机。在 Kubernetes 中，节点仅仅是一条记录，节点创建之后，Kubernetes 会检查其是否可用。在 Kubernetes 中，节点用如下结构保存：

```
{
  "id": "10.1.2.3",
  "kind": "Minion",
  "apiVersion": "v1beta1",
  "resources": {
    "capacity": {
      "cpu": 1000,
      "memory": 1073741824
    },
  },
  "labels": {
    "name": "my-first-k8s-node",
  },
}
```

Kubernetes 校验节点可用依赖于 ID。在当前的版本中，有两个接口可以用来管理节点：节点控制和 Kube 管理。

#### 节点控制
在 Kubernetes 主节点中，节点控制器是用来管理节点的组件。主要包含：
  -   集群范围内节点同步
  -   单节点生命周期管理
节点控制有一个同步轮寻，主要监听所有云平台的虚拟实例，会根据节点状态创建和删除。可以通过 `--node_sync_period`标志来控制该轮寻。如果一个实例已经创建，节点控制将会为其创建一个结构。同样的，如果一个节点被删除，节点控制也会删除该结构。在 Kubernetes 启动时可用通过 `--machines`标记来显示指定节点。同样可以使用 `kubectl` 来一条一条的添加节点，两者是相同的。通过设置 `--sync_nodes=false`标记来禁止集群之间的节点同步，你也可以使用 api/kubectl 命令行来增删节点。

### 容器组
在 Kubernetes 中，使用的最小单位是容器组，容器组是创建，调度，管理的最小单位。 一个容器组使用相同的 Docker 容器并共享卷（挂载点）。一个容器组是一个特定应用的打包集合，包含一个或多个容器。

#### 资源共享和通信
容器组主要是为了数据共享和它们之间的通信。
在一个容器组中，容器都使用相同的网络地址和端口，可以通过本地网络来相互通信。每个容器组都有独立的 IP，可用通过网络来和其他物理主机或者容器通信。
容器组有一组存储卷（挂载点），主要是为了让容器在重启之后可以不丢失数据。

#### 容器组管理
容器组是一个应用管理和部署的高层次抽象，同时也是一组容器的接口。容器组是部署、水平放缩的最小单位。

#### 容器组的使用
容器组可以通过组合来构建复杂的应用，其本来的意义包含：
  -   内容管理，文件和数据加载以及本地缓存管理等。
  -   日志和检查点备份，压缩，快照等。
  -   监听数据变化，跟踪日志，日志和监控代理，消息发布等。
  -   代理，网桥
  -   控制器，管理，配置以及更新

#### 容器组的生命状态
包括若干状态值：`pending`、`running`、`succeeded`、`failed`

## 架构设计

Kubernetes 首先是一套分布式系统，由多个节点组成，节点分为两类：一类是属于管理平面的主节点/控制节点（Master Node）；一类是属于运行平面的工作节点（Worker Node）。

![](./image/Kubernetes-framework-1.png)


### 控制平面
#### 主节点服务
主节点上需要提供如下的管理服务：
  -   `apiserver` 是整个系统的对外接口，提供一套 RESTful 的 [Kubernetes API](https://kubernetes.io/zh/docs/concepts/overview/kubernetes-api/)，供客户端和其它组件调用；
  -   `scheduler` 负责对资源进行调度，分配某个 pod 到某个节点上。是 pluggable 的，意味着很容易选择其它实现方式；
  -   `controller-manager` 负责管理控制器，包括 endpoint-controller（刷新服务和 pod 的关联信息）和 replication-controller（维护某个 pod 的复制为配置的数值）。

#### Etcd
这里 Etcd 即作为数据后端，又作为消息中间件。
通过 Etcd 来存储所有的主节点上的状态信息，很容易实现主节点的分布式扩展。
组件可以自动的去侦测 Etcd 中的数值变化来获得通知，并且获得更新后的数据来执行相应的操作。

### 工作节点
  -   kubelet 是工作节点执行操作的 agent，负责具体的容器生命周期管理，根据从数据库中获取的信息来管理容器，并上报 pod 运行状态等；
  -   kube-proxy 是一个简单的网络访问代理，同时也是一个 Load Balancer。它负责将访问到某个服务的请求具体分配给工作节点上的 Pod（同一类标签）。

![](D:\Documents\GitHub\g1wang.github.io\Blog\Docker\image\Kubernetes-framework-2.png)

## 部署 Kubernetes
部署 Kubernetes几种方式
  -   kubeadm
  -   docker-desktop
  -   k3s

### 使用 kubeadm 部署 kubernetes(CRI 使用 containerd)
`kubeadm` 提供了 `kubeadm init` 以及 `kubeadm join` 这两个命令作为快速创建 `kubernetes` 集群的最佳实践。

#### 安装 containerd
```
# rhel 系
sudo yum install containerd.io
```

#### 配置 containerd
新建 `/etc/systemd/system/cri-containerd.service` 文件
```
[Unit]
Description=containerd container runtime for kubernetes
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/containerd --config //etc/cri-containerd/config.toml

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

新建 `/etc/cri-containerd/config.toml` containerd 配置文件
```
version = 2
# persistent data location
root = "/var/lib/cri-containerd"
# runtime state information
state = "/run/cri-containerd"
plugin_dir = ""
disabled_plugins = []
required_plugins = []
# set containerd's OOM score
oom_score = 0

[grpc]
  address = "/run/cri-containerd/cri-containerd.sock"
  tcp_address = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  # socket uid
  uid = 0
  # socket gid
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216

[debug]
  address = ""
  format = "json"
  uid = 0
  gid = 0
  level = ""

[metrics]
  address = "127.0.0.1:1338"
  grpc_histogram = false

[cgroup]
  path = ""

[timeouts]
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[plugins]
  [plugins."io.containerd.gc.v1.scheduler"]
    pause_threshold = 0.02
    deletion_threshold = 0
    mutation_threshold = 100
    schedule_delay = "0s"
    startup_delay = "100ms"
  [plugins."io.containerd.grpc.v1.cri"]
    disable_tcp_service = true
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    stream_idle_timeout = "4h0m0s"
    enable_selinux = false
    selinux_category_range = 1024
    sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.5"
    stats_collect_period = 10
    # systemd_cgroup = false
    enable_tls_streaming = false
    max_container_log_line_size = 16384
    disable_cgroup = false
    disable_apparmor = false
    restrict_oom_score_adj = false
    max_concurrent_downloads = 3
    disable_proc_mount = false
    unset_seccomp_profile = ""
    tolerate_missing_hugetlb_controller = true
    disable_hugetlb_controller = true
    ignore_image_defined_volumes = false
    [plugins."io.containerd.grpc.v1.cri".containerd]
      snapshotter = "overlayfs"
      default_runtime_name = "runc"
      no_pivot = false
      disable_snapshot_annotations = false
      discard_unpacked_layers = false
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          pod_annotations = []
          container_annotations = []
          privileged_without_host_devices = false
          base_runtime_spec = ""
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            # SystemdCgroup enables systemd cgroups.
            SystemdCgroup = true
            # BinaryName is the binary name of the runc binary.
            # BinaryName = "runc"
            # BinaryName = "crun"
            # NoPivotRoot disables pivot root when creating a container.
            # NoPivotRoot = false

            # NoNewKeyring disables new keyring for the container.
            # NoNewKeyring = false

            # ShimCgroup places the shim in a cgroup.
            # ShimCgroup = ""

            # IoUid sets the I/O's pipes uid.
            # IoUid = 0

            # IoGid sets the I/O's pipes gid.
            # IoGid = 0

            # Root is the runc root directory.
            Root = ""

            # CriuPath is the criu binary path.
            # CriuPath = ""

            # CriuImagePath is the criu image path
            # CriuImagePath = ""

            # CriuWorkPath is the criu work path.
            # CriuWorkPath = ""
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      max_conf_num = 1
      conf_template = ""
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = "/etc/cri-containerd/certs.d"
      [plugins."io.containerd.grpc.v1.cri".registry.headers]
        # Foo = ["bar"]
    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = ""
    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""
  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/cri-containerd"
  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"
  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"
  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false
  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]
  [plugins."io.containerd.snapshotter.v1.devmapper"]
    root_path = ""
    pool_name = ""
    base_image_size = ""
    async_remove = false
```

#### 安装 **kubelet** **kubeadm** **kubectl** **cri-tools** **kubernetes-cni**
###### CentOS/Fedora
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


# 如果使用镜像会报错，验证的GPG key验证问题
#修改 /etc/yum.repos.d/kubernetes.repo 
#参数： repo_gpgcheck=0

sudo yum install -y kubelet kubeadm kubectl
```

#### 修改内核的运行参数
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 应用配置
sudo sysctl --system
```

#### 配置 kubelet
修改 `kubelet.service`
/etc/systemd/system/kubelet.service.d/10-proxy-ipvs.conf 写入以下内容
```
# 启用 ipvs 相关内核模块
[Service]
ExecStartPre=-/sbin/modprobe ip_vs
ExecStartPre=-/sbin/modprobe ip_vs_rr
ExecStartPre=-/sbin/modprobe ip_vs_wrr
ExecStartPre=-/sbin/modprobe ip_vs_sh
```
执行以下命令应用配置。
```
sudo systemctl daemon-reload
```

#### 部署
###### master
```
sudo systemctl enable cri-containerd
sudo systemctl start cri-containerd

sudo kubeadm init \
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
--pod-network-cidr 10.244.0.0/16 \
--cri-socket /run/cri-containerd/cri-containerd.sock \
--v 5 \
--ignore-preflight-errors=all

```

-  kubeadm init 可能报错
```
1.It seems like the kubelet isn't running or healthy

# check if swap is diabled on your node as you MUST disable swap in order for the kubelet to work properly. 

sudo swapoff -a  
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# check out if kubernetes and docker cgroup driver is set to same.
sudo docker info |grep -i cgroup

sudo vim /etc/docker/daemon.json

{
    "exec-opts": ["native.cgroupdriver=systemd"]
}

# Restart your docker service
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet
```

