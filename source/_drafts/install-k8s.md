---
title: 使用kubeadm安装kubernetes
tags:
- k8s
---

## 安装k8s

先简单记录一些命令，环境的配置另外再补充。

重置集群
```shell
$ kubeadmin reset
```

重新安装，使用`kubernetes-version`指定版本以复用已下载好的镜像，使用`-v=5`开启最高的日志输出查看是否有异常。
```shell
$ kubeadm init --kubernetes-version=v1.18.3 -v=5
```

在其他机器上加入集群

```shell
// 如果之前已经加入了集群，则需要重置
$ kubeadmin reset 
// 这个命令在上面init的最后会输出，复制输入即可
$ kubeadm join 192.168.1.135:6443 --token <need_to_fill> \
    --discovery-token-ca-cert-hash <need_to_fill>
```

复制一下`kubectl`的配置

```shell
$ cp /etc/kubernetes/admin.conf ~/.kube/config
```

## 安装rancher

自签名证书的过程另行补充。

环境配置见[这里](https://rancher.com/docs/rancher/v2.x/en/installation/install-rancher-on-k8s/)

最后的安装命令
```
helm install rancher rancher-stable/rancher \
  --version 2.4.5 \
  --namespace cattle-system \
  --set hostname=rancher.local.org \
  --set ingress.tls.source=secret \
  --set privateCA=true
```