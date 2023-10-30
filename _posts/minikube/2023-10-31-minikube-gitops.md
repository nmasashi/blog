---
layout: post
title: "minikube でGitOps"
date: 2023-10-31
categories: Kubernetes minikube
---

- TOC
{:toc}

# minikubeでGitOps

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

