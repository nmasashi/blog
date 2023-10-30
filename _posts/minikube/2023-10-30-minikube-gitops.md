---
layout: post
title: "minikube で GitOps"
date: 2023-10-30
categories: Kubernetes minikube
---

- TOC
{:toc}

# minikubeでGitOps

minikubeでArgo CDを使用してGitOpsしてみる

## 準備

minikubeを起動
```shell
$ minikube start
😄  minikube v1.31.2 on Ubuntu 20.04 (amd64)
✨  Automatically selected the docker driver
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

## Argo CDをインストール

Argo CD用のnamespaceを作成
```shell
$ kubectl create namespace argocd
namespace/argocd created
```

Argo CDに関するリソースがインストールされる
```shell
$ kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
customresourcedefinition.apiextensions.k8s.io/applications.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/applicationsets.argoproj.io created
customresourcedefinition.apiextensions.k8s.io/appprojects.argoproj.io created
serviceaccount/argocd-application-controller created
...
networkpolicy.networking.k8s.io/argocd-repo-server-network-policy created
networkpolicy.networking.k8s.io/argocd-server-network-policy created
```

podを見てみるとなんかいろいろできている
```shell
$ kubectl get pod -n argocd
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          2m27s
argocd-applicationset-controller-787bfd9669-4whvq   1/1     Running   0          2m29s
argocd-dex-server-bb76f899c-j7thv                   1/1     Running   0          2m29s
argocd-notifications-controller-5557f7bb5b-n5ff6    1/1     Running   0          2m29s
argocd-redis-b5d6bf5f5-gtsfw                        1/1     Running   0          2m28s
argocd-repo-server-56998dcf9c-v9h59                 1/1     Running   0          2m28s
argocd-server-5985b6cf6f-8v459                      1/1     Running   0          2m28s
```

## argocd cli インストール

```shell
$ curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
$ sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
$ rm argocd-linux-amd64

# 確認
$ argocd version
argocd: v2.8.4+c279299
  BuildDate: 2023-09-13T19:43:37Z
  GitCommit: c27929928104dc37b937764baf65f38b78930e59
  GitTreeState: clean
  GoVersion: go1.20.7
  Compiler: gc
  Platform: linux/amd64
FATA[0000] Argo CD server address unspecified 
```

## Argo CD APIサーバーにLBでアクセス

```shell
$ kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

$ minikube tunnel
```
## ログイン

パスワードの初期化
```shell
$ argocd admin initial-password -n argocd
RyUEreNxZbnqVb9N

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```

CLIでログインする。usernameはadmin
```shell
$ argocd login localhost
WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Username: admin
Password: 
'admin:login' logged in successfully
Context 'localhost' updated
```
証明書関係のWARNINGが出るけどいったん無視。

画面からも同じ情報でログインできる。

![]({{site.baseurl}}/images/minikube/argocd.png)

## サンプルアプリのデプロイ

サンプルアプリのimageを作成
```shell
docker build -t k8s-sample-app .
```

minikubeにサンプルアプリのimageを読み込む
```shell
minikube image load k8s-sample-app
```

argoの画面からNEW APPをクリック

![]({{site.baseurl}}/images/minikube/deploy_ui_1.png)

必要な設定を入力していく (1)

![]({{site.baseurl}}/images/minikube/deploy_ui_2.png)

必要な設定を入力していく (2)

![]({{site.baseurl}}/images/minikube/deploy_ui_3.png)

必要な設定を入力していく (3)

![]({{site.baseurl}}/images/minikube/deploy_ui_4.png)

設定後、アプリを作成するとこのようになる。まだk8sにPodなどのリソースが作成されていない（githubと同期していない）ため、OutOfSyncのステータスになっている。
![]({{site.baseurl}}/images/minikube/deploy_ui_5.png)

アプリの中身はdeploymentとserviceとconfigMapがあるのがわかる。SYNCを押してみる。
![]({{site.baseurl}}/images/minikube/deploy_ui_6.png)

特になにもせずに、SYNCHRONIZEをクリック
![]({{site.baseurl}}/images/minikube/deploy_ui_7.png)

ちゃんとGithubにあるk8sのリソースを読み込んでPodが起動した。
![]({{site.baseurl}}/images/minikube/deploy_ui_8.png)

Pod数が2になるようにGithubにあるyamlを更新してみると、arugoの画面では同期されていないことが表示されていた。
![]({{site.baseurl}}/images/minikube/deploy_ui_9.png)

同期すると2つのPodが消され始めて、
![]({{site.baseurl}}/images/minikube/deploy_ui_10.png)

最終的には2つだけになりちゃんと同期されていた。
![]({{site.baseurl}}/images/minikube/deploy_ui_11.png)


## 参考

[Argo CD getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)