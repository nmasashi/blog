---
layout: post
title: "minikube get start"
date: 2023-09-21 17:35:11 +0900
categories: Kubernetes minikube
---

minikube version: v1.31.2

[minikube 公式の手順](https://minikube.sigs.k8s.io/docs/start/)を参考にしてインストールして動かくか確認

## 環境

- Windows 11
- WSL2
- Ubuntu 20.04.6 LTS

## インストール

```sh
cd [WORK-DIR]
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 稼働確認

### バージョン確認

```shell
$ minikube version
minikube version: v1.31.2
commit: fd7ecd9c4599bef9f04c0986c4a0187f98a4396e
```

### クラスターの開始

```shell
minikube start
😄  minikube v1.31.2 on Ubuntu 20.04 (amd64)
👎  Unable to pick a default driver. Here is what was considered, in preference order:
    ▪ docker: Not healthy: "docker version --format {{.Server.Os}}-{{.Server.Version}}:{{.Server.Platform.Name}}" exit status 1: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
    ▪ docker: Suggestion: Start the Docker service <https://minikube.sigs.k8s.io/docs/drivers/docker/>
💡  Alternatively you could install one of these drivers:
    ▪ kvm2: Not installed: exec: "virsh": executable file not found in $PATH
    ▪ podman: Not installed: exec: "podman": executable file not found in $PATH
    ▪ qemu2: Not installed: exec: "qemu-system-x86_64": executable file not found in $PATH
    ▪ virtualbox: Not installed: unable to find VBoxManage in $PATH

❌  Exiting due to DRV_DOCKER_NOT_RUNNING: Found docker, but the docker service isn't running. Try restarting the docker service.
```

失敗した。
docker が起動していなかったので、起動して再度クラスター開始。

```shell
$ sudo service docker start
$ minikube start
😄  minikube v1.31.2 on Ubuntu 20.04 (amd64)
✨  Automatically selected the docker driver. Other choices: ssh, none
📌  Using Docker driver with root privileges
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.27.4 preload ...
    > gcr.io/k8s-minikube/kicbase...:  447.62 MiB / 447.62 MiB  100.00% 2.96 Mi
    > preloaded-images-k8s-v18-v1...:  393.21 MiB / 393.21 MiB  100.00% 2.52 Mi
🔥  Creating docker container (CPUs=2, Memory=2200MB) ...
🐳  Preparing Kubernetes v1.27.4 on Docker 24.0.4 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔗  Configuring bridge CNI (Container Networking Interface) ...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
💡  kubectl not found. If you need it, try: 'minikube kubectl -- get pods -A'
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```

kubectl が必要なら`minikube kubectl -- get pods -A`を実行してと書かれているので実行

```shell
$ minikube kubectl -- get po -A
    > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
    > kubectl:  46.98 MiB / 46.98 MiB [--------------] 100.00% 1.00 MiB p/s 47s
NAMESPACE     NAME                               READY   STATUS    RESTARTS      AGE
kube-system   coredns-5d78c9869d-g89th           1/1     Running   0             37m
kube-system   etcd-minikube                      1/1     Running   0             38m
kube-system   kube-apiserver-minikube            1/1     Running   0             38m
kube-system   kube-controller-manager-minikube   1/1     Running   0             38m
kube-system   kube-proxy-8kl7r                   1/1     Running   0             37m
kube-system   kube-scheduler-minikube            1/1     Running   0             38m
kube-system   storage-provisioner                1/1     Running   1 (37m ago)   38m
```

コマンドがインストールと pod の情報が取得できた。

### ダッシュボードの起動

```shell
minikube dashboard --url
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
http://127.0.0.1:43057/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/
```

初回は dashbord 用の pod が立ち上がるまで 10 分くらいかかった。

こんな感じでダッシュボードが立ち上がる。

![image]({{site.baseurl}}/images/minikube/dashbord.png)

## アプリケーションのデプロイ

### サービス版

#### deployment の作成

```shell
$ kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
deployment.apps/hello-minikube created

$ kubectl get deployment -n default
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1/1     1            1           46s
```

#### service の作成

```shell
$ kubectl expose deployment hello-minikube --type=NodePort --port=8080
service/hello-minikube exposed

$ kubectl get service -n default
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.110.105.101   <none>        8080:31167/TCP   29s
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP          16h
```

kubernetes という名前のサービスは最初から作られていたっぽい

#### アクセス

minikube のコマンドを使用する方法

```shell
minikube service hello-minikube
```

kubectl でポートフォワードを使用して接続する方法

```shell
$ kubectl port-forward service/hello-minikube 7080:8080
Forwarding from 127.0.0.1:7080 -> 8080
Forwarding from [::1]:7080 -> 8080
Handling connection for 7080
Handling connection for 7080
```

ブラウザでアクセスするとこんな画面が表示される
![image]({{site.baseurl}}/images/minikube/sampleapp.png)

#### 掃除

```shell
$ kubectl delete services hello-minikube -n default
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube -n default
deployment.apps "hello-minikube" deleted
```

### ロードバランサー版

#### deployment の作成

```shell
$ kubectl create deployment balanced --image=kicbase/echo-server:1.0
deployment.apps/balanced created
```

#### service の作成

```shell
$ kubectl expose deployment balanced --type=LoadBalancer --port=8080
service/balanced exposed

$ kubectl get services -n default
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
balanced     LoadBalancer   10.102.160.89   <pending>     8080:31180/TCP   2m17s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          17h
```

ロードランサーはできているけど EXTERNAL-IP は pending のまま

#### アクセス

```shell
$ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

🏃  Starting tunnel for service balanced.
```

`minikube tunnel`を流しているものとは別のターミナルでサービスの様子を見てみる

```shell
$ kubectl get services -n default
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
balanced     LoadBalancer   10.102.160.89   127.0.0.1     8080:31180/TCP   3m57s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          17h
```

EXTERNAL-IP に ip アドレスが設定されていた。`http://127.0.0.1:8080/`でアクセスすることができた。

#### 掃除

```shell
$ kubectl delete deployment balanced
deployment.apps "balanced" deleted
$ kubectl delete service balanced -n default
service "balanced" deleted
```

### イングレス版

#### アドオンの有効化

```shell
$ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
    ▪ Using image registry.k8s.io/ingress-nginx/controller:v1.8.1
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
    ▪ Using image registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
```

#### サンプルの適用

```shell
$ kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
pod/foo-app created
service/foo-service created
pod/bar-app created
service/bar-service created
ingress.networking.k8s.io/example-ingress created
```

作成されたリソースの確認

```shell
$ kubectl get pod,svc,ingress -n default
NAME          READY   STATUS    RESTARTS   AGE
pod/bar-app   1/1     Running   0          113s
pod/foo-app   1/1     Running   0          113s

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/bar-service   ClusterIP   10.98.118.218   <none>        8080/TCP   113s
service/foo-service   ClusterIP   10.110.10.101   <none>        8080/TCP   113s
service/kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    24h

NAME                                        CLASS   HOSTS   ADDRESS        PORTS   AGE
ingress.networking.k8s.io/example-ingress   nginx   *       192.168.49.2   80      113s
```

pod と service が 2 つづつできている。あとは ingress が作成されている。

作成したリソースの詳細はこんな感じ。

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
    - name: foo-app
      image: "kicbase/echo-server:1.0"
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
    - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
    - name: bar-app
      image: "kicbase/echo-server:1.0"
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
    - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /foo
            backend:
              service:
                name: foo-service
                port:
                  number: 8080
          - pathType: Prefix
            path: /bar
            backend:
              service:
                name: bar-service
                port:
                  number: 8080
---
```

pod とそれに紐づく service が作成されていて、その service に接続できるパスが ingress に設定されている。

#### アクセス確認

アドレス確認

```shell
$ kubectl get ingress -n default
NAME              CLASS   HOSTS   ADDRESS        PORTS   AGE
example-ingress   nginx   *       192.168.49.2   80      15m
```

アクセスしてみる

```shell
$ curl 192.168.49.2/foo
Request served by foo-app

HTTP/1.1 GET /foo

Host: 192.168.49.2
Accept: */*
User-Agent: curl/7.68.0
X-Forwarded-For: 192.168.49.1
X-Forwarded-Host: 192.168.49.2
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Scheme: http
X-Real-Ip: 192.168.49.1
X-Request-Id: 0b6366f629e56aec540ca5236c712336
X-Scheme: http
```

```shell
$ curl 192.168.49.2/bar
Request served by bar-app

HTTP/1.1 GET /bar

Host: 192.168.49.2
Accept: */*
User-Agent: curl/7.68.0
X-Forwarded-For: 192.168.49.1
X-Forwarded-Host: 192.168.49.2
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Scheme: http
X-Real-Ip: 192.168.49.1
X-Request-Id: de3be4e874816bf12a22b2550edb325b
X-Scheme: http
```

#### 掃除

```shell
$ kubectl delete -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
pod "foo-app" deleted
service "foo-service" deleted
pod "bar-app" deleted
service "bar-service" deleted
ingress.networking.k8s.io "example-ingress" deleted
```

## クラスターの停止

```shell
$ minikube stop
✋  Stopping node "minikube"  ...
🛑  Powering off "minikube" via SSH ...
🛑  1 node stopped.
```
