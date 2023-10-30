---
layout: post
title: "minikube Multi-Node Clusters"
date: 2023-10-15
categories: Kubernetes minikube
---

- TOC
{:toc}

# Multi-Node Clusters

minikube をマルチノードにしてみていろいろしてみる

## やりたいこと

![]({{site.baseurl}}/images/minikube/multi-clusters.png)

1. minikube をマルチノード構成にする
2. ローカルの自作 docker image を minikube で動かす
3. ちゃんと負荷分散しているか見てみる

## minikube をマルチノード構成にする

minikube を起動するときに node 数を指定する。

```shell
$ minikube start --nodes 2 -p multinode-demo
😄  [multinode-demo] minikube v1.31.2 on Ubuntu 20.04 (amd64)
✨  Automatically selected the docker driver. Other choices: none, ssh
📌  Using Docker driver with root privileges
👍  Starting control plane node multinode-demo in cluster multinode-demo
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass

👍  Starting worker node multinode-demo-m02 in cluster multinode-demo
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🌐  Found network options:
    ▪ NO_PROXY=192.168.58.2
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ env NO_PROXY=192.168.58.2
🔎  Verifying Kubernetes components...
🏄  Done! kubectl is now configured to use "multinode-demo" cluster and "default" namespace by default
```

ノードを確認すると二つ動いている

```shell
$ kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
multinode-demo       Ready    control-plane   47s   v1.27.4
multinode-demo-m02   Ready    <none>          29s   v1.27.4
```

ちなみに、docker コンテナはこんな感じになっている

```shell
$ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED              STATUS              PORTS
              NAMES
b773c69e6a24   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   127.0.0.1:32777->22/tcp, 127.0.0.1:32776->2376/tcp, 127.0.0.1:32775->5000/tcp, 127.0.0.1:32774->8443/tcp, 127.0.0.1:32773->32443/tcp   multinode-demo-m02
7eb23f0b5ea8   gcr.io/k8s-minikube/kicbase:v0.0.40   "/usr/local/bin/entr…"   About a minute ago   Up About a minute   127.0.0.1:32772->22/tcp, 127.0.0.1:32771->2376/tcp, 127.0.0.1:32770->5000/tcp, 127.0.0.1:32769->8443/tcp, 127.0.0.1:32768->32443/tcp   multinode-demo
```

## ローカルの自作 docker image を minikube で動かす

### minikube でローカルの docker image を読み込む

node 名と pod 名を出力するものを実装 ([k8s-environment-variable - github](https://github.com/nmasashi/k8s-environment-variable/tree/main))

```shell
$ cd [k8s-environment-variableをcloneしたフォルダ]

# image build
$ docker build -t k8s-environment-variable .

# k8s-environment-variable:latestができたらOK
$ docker images
```

minikube で使用できるように image を読み込む。

```shell
$ minikube image load k8s-environment-variable -p multinode-demo
```

読み込めたか確認する。`docker.io/library/k8s-environment-variable:latest`が読み込んだ image。

```shell
$ minikube image ls -p multinode-demo
registry.k8s.io/pause:3.9
registry.k8s.io/kube-scheduler:v1.27.4
registry.k8s.io/kube-proxy:v1.27.4
registry.k8s.io/kube-controller-manager:v1.27.4
registry.k8s.io/kube-apiserver:v1.27.4
registry.k8s.io/etcd:3.5.7-0
registry.k8s.io/coredns/coredns:v1.10.1
gcr.io/k8s-minikube/storage-provisioner:v5
docker.io/library/k8s-environment-variable:latest
docker.io/kindest/kindnetd:v20230511-dc714da8
```

読み込んだ image から deployment を作成。
deployment は pod を 4 つ配置するように設定してある。

```shell
$ kubectl apply -f k8s/deployment.yaml
deployment.apps/k8s-environment-variable-deployment created
```

ちゃんと動いているっぽい

```shell
$ kubectl get pod
NAME                                                   READY   STATUS    RESTARTS   AGE
k8s-environment-variable-deployment-577848df7d-7nptw   1/1     Running   0          3s
k8s-environment-variable-deployment-577848df7d-pkc4d   1/1     Running   0          3s
k8s-environment-variable-deployment-577848df7d-s75fj   1/1     Running   0          3s
k8s-environment-variable-deployment-577848df7d-vvlz6   1/1     Running   0          3s
```

それぞれの node で pod が 2 つずつ作成されている。

```shell
$ kubectl get pods -o wide |  awk -v 'OFS=\t' '{print $1,$7}'
NAME    NODE
k8s-environment-variable-deployment-577848df7d-7nptw    multinode-demo-m02
k8s-environment-variable-deployment-577848df7d-pkc4d    multinode-demo
k8s-environment-variable-deployment-577848df7d-s75fj    multinode-demo
k8s-environment-variable-deployment-577848df7d-vvlz6    multinode-demo-m02
```

LoadBalancer を作成する。

```shell
$ kubectl expose deployment k8s-environment-variable-deployment --type=LoadBalancer --port=8080
service/k8s-environment-variable-deployment exposed
```

localhost で接続できるようにする。

```shell
$ minikube tunnel -p multinode-demo
```

こんな感じで node 名と pod 名が返ってくる。

```shell
$ curl localhost:8080
{"node_name":"multinode-demo","pod_name":"k8s-environment-variable-deployment-577848df7d-s75fj"}
```

## ちゃんと負荷分散しているか見てみる

1000 回アクセスしてみる。

```shell
$ for i in {1..1000} ; do curl localhost:8080 ; done > output.txt
```

集計

<table border=1 >
  <tr>
    <th>node 名</th>
    <th>アクセス回数</th>
    <th>pod 名（サフィックス）</th>
    <th>アクセス回数</th>
  </tr>
  <tr>
    <td rowspan="2">multinode-demo</td>
    <td rowspan="2" align="center">498</td>
    <td>s75fj</td>
    <td align="center">262</td>
  </tr>
  <tr>
    <td>pkc4d</td>
    <td align="center">236</td>
  </tr>
  <tr>
    <td rowspan="2">multinode-demo-m02</td>
    <td rowspan="2" align="center">502</td>
    <td>7nptw</td>
    <td align="center">274</td>
  </tr>
  <tr>
    <td>vvlz6</td>
    <td align="center">228</td>
  </tr>
</table>

いい感じに分散されてそう。

## 参考

[minikube 公式ページ](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/)
