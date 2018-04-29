title: Kubernetes集群之路（一）TLS证书配置
date: 2018-3-26 12:40:08
tags:

- docker
- kubernetes

---

![](https://upload.wikimedia.org/wikipedia/commons/6/67/Kubernetes_logo.svg)

{%note default%}

Kubernetes是Google开源的容器化集群管理系统，其提供的应用部署、扩展、服务发现等机制对于微服务化架构应用有着十分重要的作用。

本系列文章基于以下版本来讲述如何使用二进制方式安装Kubernetes集群顺便讲述下踩坑的心路历程：
 - Kubernetes version: `v1.10`
 - System: `CentOS Linux 7`
 - Kernel: `Linux 3.10.0`

{%endnote%}
Kubernetes系统的各个组件需要使用`TLS`证书对其通信加密以及授权认证，所以在部署之前我们需要先生成相关的`TLS`证书以便后续操作能够顺利进行。

<!--more-->

-----------------

{%note danger%}
在后续安装部署中，将不使用`kube-apiserver`的HTTP非安全端口，所有组件都启用`TLS`双向认证通信。因此`TLS`证书配置是在安装配置Kubernetes系统中最容易出错和难于排查问题的一步，所以请务必耐心仔细。
{%endnote%}


在开始前，为了模拟集群节点，我们假定需要在以下三台Linux主机上部署Kubernetes:

 - `10.138.148.161 `：作为`master&worker`节点
 - `10.138.196.180`：作为`worker`节点
 - `10.138.212.68`：作为`worker`节点


------------------

### 安装`CFSSL`证书生成工具

{%note info%}
我们将使用`Cloudflare`的PKI工具集[cloudflare/cfssl](https://github.com/cloudflare/cfssl)来生成集群所需要的各种`TLS`证书。
{%endnote%}

执行以下命令直接下载二进制文件进行安装:

```bash
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 -O cfssl 
chmod +x cfssl 
sudo mv cfssl /usr/local/bin

wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 -O cfssljson 
chmod +x cfssljson 
sudo mv cfssljson /usr/local/bin

wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64 -O cfssl-certinfo 
chmod +x cfssl-certinfo 
sudo mv cfssl-certinfo /usr/local/bin

export PATH=/usr/local/bin:$PATH
```

### 创建CA根证书（Certificate Authority）

CA（Certificate Authority）是自签名的根证书，用来签名后续创建的其它 TLS 证书；
确认`CFSSL`工具安装成功之后，我们先通过`CFSSL`工具来创建模版配置json文件:

```bash
cfssl print-defaults config > config.json
cfssl print-defaults csr > csr.json
```
#### 创建CA配置文件

这将生成两个模版json文件，后续`CFSSL`将读取json文件内容并生成对应的`pem`文件。我们先复制`config.json`为`ca-config.json`文件并做如下修改:

```json
{
    "signing": {
        "default": {
            "expiry": "99999h"
        },
        "profiles": {
            "kubernetes": {
                "expiry": "99999h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
```

{%note warning%}
`profiles`：可以定义多个profiles，分别指定不同的过期时间、使用场景等参数；后续在签名证书时使用某个特定的profile。

`signing`：表示该证书可用于签名(签发)其它证书，生成的 ca.pem 证书中 CA=TRUE。

`server auth`：表示`client可以用该 CA（生成的ca.pem） 对server提供的证书进行验证。

`client auth`：表示server可以用该CA(生成的ca.pem）对client提供的证书进行验证。
{%endnote%}



#### 创建CA证书签名请求

我们复制`csr.json`为`ca-csr.json`并做以下修改:

```json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```



{%note warning%}

`CN`(`Common Name`):后续`kube-apiserver`组件将从证书中提取该字段作为请求的用户名；

`O`(`Organtzation`):后续`kube-apiserver`组件将从证书中提取该字段作为请求的用户所属的用户组；

{%endnote%}

#### 生成CA证书和私钥

执行以下命令来生成CA证书和私钥：

```bash
cfssl gencert -initca ca-csr.json | cfssljson -bare ca
ls ca*
ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

这样，我们就生成了CA证书和私钥了，因为我们需要双向`TLS`认证，所以需要拷贝`ca-key.pem`和`ca.pem`到所有要部署的机器的`/etc/kubernetes/ssl`目录下备用。



### 创建kubernetes组件认证授权证书

因为我们准备部署的kubernetes组件是使用`TLS`双向认证的，包括`kube-apiserver`不打算使用HTTP端口，因此，我们需要生成以下的证书以供后续组件部署的时候备用：



{%note info%}

- `etcd`证书：etcd集群之间通信加密使用的`TLS`证书。
- `kube-apiserver`证书：配置`kube-apiserver`组件的证书。
- `kube-controller-manager`证书：用于和`kube-apiserver`通信认证的证书。
- `kube-scheduler`证书：用于和`kube-apiserver`通信认证的证书。
- `kubelet`证书【可选,非必需】：用于和`kube-apiserver`通信认证的证书，如果使用`TLS Bootstarp`认证方式，将没有必要配置。
- `kube-proxy`证书【可选,非必需】：用于和`kube-apiserver`通信认证的证书，如果使用`TLS Bootstarp`认证方式，将没有必要配置。

{%endnote%}

下面我们将逐个创建对应的`TLS`证书，并做相应的简短说明：

#### 创建`etcd`证书：

首选我们创建`etcd`证书签名请求(CSR)，拷贝`csr.json`为`etcd-csr.json`并做以下修改:

```json
{
  "CN": "etcd",
    "hosts": [
      "127.0.0.1",
      "10.138.212.68",
      "10.138.196.180",
      "10.138.148.161",
        "master",
        "node1",
        "node2"
    ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

{%note danger%}

此处需要指定`host`字段的值，该值为所有需要部署etcd节点的`ip 域名 或者 hostname`，etcd需要使用`Subject Alternative Name（SAN）`来校验集群以及防止滥用。如果你不清楚应该使用哪个ip，默认情况下使用`ip a`查看`eth0`即可。此处指定的`ip`与后续指定的`etcd的systemd`配置`initial-cluster`相关。

相关阅读: [Option to accept TLS client certificates even if they lack correct Subject Alternative Names](https://github.com/coreos/etcd/issues/2056)

{%endnote%}

生成`etcd`证书和私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
ls etcd*
etcd.csr  etcd-csr.json  etcd-key.pem etcd.pem
```

将生成的`etch-key.pem`和`etcd.pem`拷贝到所有需要部署`etcd`集群的服务器`/etc/etcd/ssl`目录下备用。

#### 创建`kube-apiserver`证书

创建`kube-apiserver`证书签名请求配置文件，拷贝`csr.json`为`kubernetes-csr.json`并做以下修改：

```json
{
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "10.138.212.68",
      "10.138.196.180",
      "10.138.148.161",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shanghai",
            "L": "Shanghai",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
```



{%note warning%}

此处指定了`host`字段来表示授权使用该证书的`ip或域名`列表，因此上述配置文件指定了要部署的kubernetes三台服务器ip（实际上只需要指定打算部署master节点的ip即可）以及`kube-apiserver`注册的名为`kubernetes`服务的服务ip（一般默认为后续配置`kube-apiserve`组件的时候指定的`—service-cluster-ip-range`网段的第一个ip。）如果你不清楚怎么操作，可以留空`host`字段。

如果你指定了`host`字段，这里如果有 `VIP` 的，也是需要填写的。

{%endnote%}

生成`kube-apiserver`证书和私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json | cfssljson -bare apiserver
ls apiserver*
apiserver.csr  apiserver-key.pem  apiserver.pem
```

我们将该证书拷贝到需要部署到`master`节点上的`/etc/kubernetes/ssl`上备用。

{%note info%}

因为我们master节点的组件之间的通信使用`非HTTP`的安全端口，所以同样也需要`TLS`认证授权，因此我们也需要配置`kube-controller-manager`和`kube-scheduler`的证书来供这两个组件访问`kube-apiserver`.如果你的集群master节点组件使用HTTP非安全端口通信，那么可以不需要配置这两个证书。

{%endnote%}

#### 创建`kube-controller-manager`证书

复制`car.json`为`kube-controller-manager-csr.json`并做以下修改：



```json
{
  "CN": "system:kube-controller-manager",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```



上述的配置中，`kube-apiserver`将提取`CN`作为客户端组件(kube-controller-manager)的用户名(system:kube-controller-manager)，`kube-apiserver`预定义的RBAC使用ClusterRoleBinding `system:kube-controller-manager`将`用户system:kube-controller-manager`与`ClusterRole system:kube-controller-manager`绑定。

生成`kube-controller-manager`证书和私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare controller-manager
ls controller-manager*
controller-manager.csr  controller-manager-key.pem  controller-manager.pem
```

将证书拷贝到需要部署`kube-controller-manager`的master节点`/etc/kubernetes/ssl`上备用。

#### 创建kube-scheduler`证书

与`kube-controller-manager`一样，`kube-scheduler`同样也需要`TLS`证书来访问`kube-apiserver`。此处不再赘述。直接上`kube-scheduler-csr.json`文件内容：

```json
{
  "CN": "system:kube-scheduler",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

``kube-scheduler `将提取 `CN作为客户端的用户名`,这里是 `system:kube-scheduler`。 kube-apiserver 预定义的 RBAC 使用的 ClusterRoleBindings `system:kube-scheduler `将`用户system:kube-scheduler `与 `ClusterRole system:kube-scheduler `绑定。

生成`kube-scheduler`证书以及私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare scheduler
ls scheduler*
scheduler.csr  scheduler-key.pem  scheduler.pem
```

将证书拷贝到需要部署`kube-scheduler`的master节点`/etc/kubernetes/ssl`上备用。

至此，`master`节点上的证书生成就全部完成了，接下来是生成`worker`节点的证书，需要注意的是：生成`worker`证书是可选的，如果你使用`TLS Bootstarpping`那么你可以跳过以下步骤`worker`证书生成工作。直接转到部署的实际操作环节。关于`TLS`证书和`TLS Bootstarpping`认证方式的区别，后续考虑单独写一遍文章展开来讲。

-------

#### 创建`kubelet`证书

拷贝`car.json`为`kubelet-csr.json`并做以下修改:

```json
{
  "CN": "system:node:node",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "system:nodes",
      "OU": "System"
    }
  ]
}
```

{%note warning%}

`O`为用户组，kubernetes RBAC定义了ClusterRoleBinding将Group system:nodes和CLusterRole system:node关联起来。

注意:在`kubernetes v1.8+`以上版本，将不会自动创建`binding`,因此我们后续需要手动创建绑定关系。

{%endnote%}

生成`kubelet`证书和私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubelet-csr.json | cfssljson -bare kubelet
ls kubelet*
kubelet.csr  kubelet-csr.json  kubelet-key.pem  kubelet.pem
```

将生成的证书和秘钥拷贝到所有需要部署的worker节点上的`/etc/kubernetes/ssl`下备用。

#### 创建`kube-proxy`证书

拷贝`car.json`为`kube-proxy-csr.json`并做以下修改:

```json
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shanghai",
      "L": "Shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
```

{%note warning%}

`CN` 指定该证书的 User为 system:kube-proxy。Kubernetes RBAC定义了ClusterRoleBinding将`system:kube-proxy用户`与`system:node-proxier 角色`绑定。system:node-proxier具有kube-proxy组件访问ApiServer的相关权限。

{%endnote%}

生成`kube-proxy`证书和私钥：

```bash
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
ls kube-proxy*
kube-proxy.csr  kube-proxy-csr.json  kube-proxy-key.pem  kube-proxy.pem
```

将生成的证书和私钥拷贝到所有需要部署`worker`节点的`/etc/kubernetes/ssl`下备用。

在完成证书分发之后，这样我们的证书相关的生成工作就完成了。接下来开始配置各个组件。



参考资料：

- [Using RBAC Authorization](https://kubernetes.io/docs/admin/authorization/rbac/)
- [Kubernetes HA Cluster Build](https://wiki.shileizcc.com/display/KUB/Kubernetes+HA+Cluster+Build)
- [在CentOS上部署kubernetes集群](https://jimmysong.io/kubernetes-handbook/practice/install-kubernetes-on-centos.html)




