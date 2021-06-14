# How to create a test env ?

## simple k8s cluster using kubeadm

+ create a centos 7 guest os using cloud vm or local vm
+ make sure your vm or host has network access to google.com
+ use this script below to install kubeadm, thanks for [link](https://gist.github.com/manics/dd44bdd78f9cc51270f8d159a5deaa87)
  ```bash
  #!/bin/bash

  set -x

  if [ -f /etc/debian_version ]; then
      apt-get update
      apt-get upgrade -y
      apt-get install -y apt-transport-https docker.io

      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
      cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
  deb http://apt.kubernetes.io/ kubernetes-xenial main
  EOF

      apt-get update
      apt-get install -y kubelet kubeadm kubectl
  fi

  if [ -f /etc/redhat-release ]; then
      yum upgrade -y
      yum install -y docker
      systemctl enable docker
      systemctl start docker
      swapoff -a

      cat <<EOF >  /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-iptables = 1
  EOF
      sysctl --system

      cat <<EOF > /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
  EOF

      setenforce 0
      yum install -y kubelet kubeadm kubectl
      systemctl enable kubelet
      systemctl start kubelet
  fi

  kubeadm init --pod-network-cidr=10.244.0.0/16

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

  kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml

  kubectl taint nodes --all node-role.kubernetes.io/master-kubectl taint nodes --all node-role.kubernetes.io/master-

  ```
+ install apps (e.g. nginx)
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    selector:
      matchLabels:
        app: nginx
    replicas: 2 # tells deployment to run 2 pods matching the template
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
          - containerPort: 80

  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: NodePort
    selector:
      app: nginx
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30080
  ```
+ deploy an ingress controller
  ```bash
  kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml
  ```
+ change your app using ingress (For more [details](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/))
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
  spec:
    selector:
      matchLabels:
        app: nginx
    replicas: 2 # tells deployment to run 2 pods matching the template
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
          - containerPort: 80

  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    selector:
      app: nginx
    ports:
      - protocol: TCP
        port: 80
        targetPort: 80
  ---
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: nginx-app-ingress
  spec:
    defaultBackend:
      service:
        name: nginx-service
        port:
          number: 80
  ```
