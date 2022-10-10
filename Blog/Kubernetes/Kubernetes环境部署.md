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
sudo mkdir -p  /etc/cri-containerd
sudo vim /etc/cri-containerd/config.toml
```

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
sudo mkdir -p /etc/systemd/system/kubelet.service.d/
sudo vim /etc/systemd/system/kubelet.service.d/10-proxy-ipvs.conf
```

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

### node 工作节点
在 **另一主机** 重复 **部署** 小节以前的步骤，安装配置好 kubelet。根据提示，加入到集群

```
sudo systemctl enable cri-containerd

sudo systemctl start cri-containerd

sudo kubeadm join 192.168.86.75:6443 \
    --token cz81zt.orsy9gm9v649e5lf \
    --discovery-token-ca-cert-hash sha256:5edb316fd0d8ea2792cba15cdf1c899a366f147aa03cba52d4e5c5884ad836fe \
    --cri-socket /run/cri-containerd/cri-containerd.sock
```

### 部署 Dashboard
[Kubernetes Dashboard](https://github.com/kubernetes/dashboard) 是基于网页的 Kubernetes 用户界面
#### 部署
执行以下命令即可部署 Dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```