title: Kubernetes集群之路（二）etcd集群部署
date: 2018-4-26 12:40:08
tags:

- docker
- kubernetes

---

{%note info%}

在前面我们生成了所有kubernetes相关的TLS证书，kubernetes集群自身所有配置相关信息都存储在`etcd`之中，`flannel`为集群中节点的`pod`提供了加入同一局域网的能力。因此接下来我们安装部署`etcd`集群以及`flannel`网络插件。{%endnote%}

因为`flannel`插件也依赖于`etcd`存储信息，所以我们首先需要安装`etcd`集群，使之实现高可用。

在开始之前请确保在上一篇文章中生成的TLS证书都分发到需要部署的`所有机器节点`的以下位置：

1. `/etc/kubernetes/ssl/etcd.pem`
2. `/etc/kubernetes/ssl/etcd-key.pem`
3. `/etc/kubernetes/ssl/ca.pem`

<!--more-->

#### 部署`etcd`

我们采用纯二进制安装`etcd`,因此不使用默认的包管理器中的安装文件。在每台需要部署的`etcd`的节点上，下载最新版本的`etcd`二进制安装包：

- [https://github.com/coreos/etcd/releases](https://github.com/coreos/etcd/releases)



##### 下载安装二进制文件

目前最新版本是[v3.3.4](https://github.com/coreos/etcd/releases/tag/v3.3.4)，找到对应的系统架构并直接下载:  [https://github.com/coreos/etcd/releases/download/v3.3.4/etcd-v3.3.4-linux-amd64.tar.gz](https://github.com/coreos/etcd/releases/download/v3.3.4/etcd-v3.3.4-linux-amd64.tar.gz)

在所有需要安装etcd节点执行以下命令来安装`etcd和etcdctl`(注意，截止到目前最新版本是v3.3.4，根据你的需要自行替换对应架构和版本):

```bash
wget https://github.com/coreos/etcd/releases/download/v3.3.4/etcd-v3.3.4-linux-amd64.tar.gz
tar zxvf etcd-v3.3.4-linux-amd64.tar.gz
cd etcd-v3.3.4-linux-amd64
sudo mv etcd etcdctl /usr/local/bin/
```



##### 配置systemd

接着，我们需要编辑对应的systemd service文件，我们需要新建一个`etcd.service`文件并放置于以下路径：`/usr/lib/systemd/system/etcd.service`并键入以下内容：

```properties
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --cert-file=${ETCD_TRUST_CERT_FILE} \
  --key-file=${ETCD_TRUST_CERT_KEY} \
  --peer-cert-file=${ETCD_TRUST_CERT_FILE} \
  --peer-key-file=${ETCD_TRUST_CERT_KEY} \
  --trusted-ca-file=${ETCD_TRUST_CA_FILE} \
  --peer-trusted-ca-file=${ETCD_TRUST_CA_FILE} \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS} \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster ${ETCD_CLUSTER_NODE_LIST} \
  --initial-cluster-state new \
  --data-dir=${ETCD_DATA_DIR}
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```



{%note warning%}

1. `WorkingDirectory`:指定 `etcd` 的工作目录和数据目录为 `/var/lib/etcd`，需在启动服务前创建这个目录。
2. 为了保证通信安全，需要指定 etcd 的公私钥(cert-file和key-file)、Peers 通信的公私钥和CA证书(peer-cert-file、peer-key-file、peer-trusted-ca-file)、客户端的CA证书（trusted-ca-file）。
3. `--initial-cluster-state` 值为 `new` 时，`--name` 的参数值必须位于 `--initial-cluster` 列表中。
4. 我们将其中一些参数的设置抽取为环境变量，以便于我们修改参数的时候不需要再次`systemctl daemon-reload`。

{%endnote%}

对应的，我们在`/etc/etcd/etcd.conf`路径中新建一个`etcd.conf`文件并键入以下内容:

```properties
# [member]
ETCD_NAME=node1
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="https://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="https://0.0.0.0:2379"

ETCD_TRUST_CA_FILE="/etc/kubernetes/ssl/ca.pem"
ETCD_TRUST_CERT_FILE="/etc/kubernetes/ssl/etcd.pem"
ETCD_TRUST_CERT_KEY="/etc/kubernetes/ssl/etcd-key.pem"

#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.138.148.161:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://10.138.148.161:2379"

ETCD_CLUSTER_NODE_LIST="node1=https://10.138.148.161:2380,node2=https://10.138.196.180:2380,node3=https://10.138.212.68:2380"
```



{%note warning%}

1. 这是节点`10.138.148.161`的环境变量配置文件内容，对于其他节点，修改对应的`ETCD_NAME`为对应的`node1、node2、node3`，并将`ETCD_INITIAL_ADVERTISE_PEER_URLS`和`ETCD_INITIAL_ADVERTISE_PEER_URLS`修改为对应的节点的`ip`。
2. 此处需要特别说明的是：`ETCD_CLUSTER_NODE_LIST`中的`ip`必须在生成etcd TLS证书时在`etcd-csr.json`中的`hosts`字段中指定（`Subject Alternative Name（SAN）`），否则可能会得到`(error "remote error: tls: bad certificate", ServerName "")`这样的错误。
3. 所有需要加入的节点都需要在`ETCD_CLUSTER_NODE_LIST`中指定，并正确配置起`ETCD_NAME`。

{%endnote%}

------

##### 验证etcd安装

在所有的节点上完成了上述两步之后，我们分别执行以下命令来启动etcd:

```bash
sudo systemctl daemon-reload
sudo systemctl start etcd
```

如果配置正确，那么上述命令执行结果应该是任何输出的。如果结果有错，请参照上述配置和环境变量文件检查配置。一旦我们顺利启动`etcd`服务，我们还需要正确检查我们的`etcd`集群是否可用，在`etcd`集群中任一节点中执行以下命令：

```bash
 etcdctl --endpoint https://127.0.0.1:2379   \
 --ca-file=/etc/kubernetes/ssl/ca.pem        \
 --cert-file=/etc/kubernetes/ssl/etcd.pem    \
 --key-file=/etc/kubernetes/ssl/etcd-key.pem \
 cluster-health
```

在一切正常情况下，你会得到类似如下的输出结果:

```bash
member 245a74588a3e85d0 is healthy: got healthy result from https://xxx.xxx.xxx.xxx:2379
member 953bacee4a009939 is healthy: got healthy result from https://xxx.xxx.xxx.xxx:2379
member f43a05b0ce1a8ed6 is healthy: got healthy result from https://xxx.xxx.xxx.xxx:2379
cluster is healthy
```

至此，我们的`etcd`集群已经顺利安装完成。接下来安装flannel插件。



#### 参考资料

- [在CentOS上部署kubernetes集群](https://jimmysong.io/kubernetes-handbook/practice/install-kubernetes-on-centos.html)