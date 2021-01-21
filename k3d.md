# Kubernetes (K3D + k3s)

- Install kubectl:
```text
$ sudo apt update && sudo apt install -y apt-transport-https gnupg2 curl

$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

$ sudo apt update

$ sudo apt install -y kubectl
```

- Install k3d
```text
$ curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | TAG=v4.0.0 bash
```
