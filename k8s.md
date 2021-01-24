# Kubernetes

## kubectl

Install kubectl:
```text
$ sudo apt update && sudo apt install -y apt-transport-https gnupg2 curl

$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

$ sudo apt update

$ sudo apt install -y kubectl
```

## kind

[kind](https://kind.sigs.k8s.io) is a tool for running local Kubernetes clusters using Docker container “nodes”.

Install kind:
```text
$ basename $(curl -fs -o/dev/null -w %{redirect_url} https://github.com/kubernetes-sigs/kind/releases/latest)
v0.10.0

$ dpkg --print-architecture
arm64

$ curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/download/v0.10.0/kind-linux-arm64
$ chmod +x ./kind
$ sudo install -m 755 -o root -g root ./kind /usr/local/bin
$ source <(kind completion bash)

$ kind --version
kind version 0.10.0
```

Create cluster:
```text
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.20.2) 🖼
 ✓ Preparing nodes 📦
 ✗ Writing configuration 📜
ERROR: failed to create cluster: failed to generate kubeadm config content: failed to get kubernetes version from node: failed to get file: command "docker exec --privileged kind-control-plane cat /kind/version" failed with error: exit status 1
Command Output: Error response from daemon: Container 8852df9c46c41ed7aeba2a8f2ac6d2208bbf920933bd3bbb54616ef09a2767ef is not running
```

It fails, because at the time of writing `kind` [does not provide](https://hub.docker.com/r/kindest/node/tags) arm64 docker images, see [here](https://github.com/kubernetes-sigs/kind/issues/166). You can use arm64 image from [here](https://hub.docker.com/r/rossgeorgiev/kind-node-arm64/)
```text
$ kind create cluster --image rossgeorgiev/kind-node-arm64:v1.20
Creating cluster "kind" ...
 ✓ Ensuring node image (rossgeorgiev/kind-node-arm64:v1.20) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
```

```text
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:37033
KubeDNS is running at https://127.0.0.1:37033/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

kind is perfect for fast testing things, but is not ment for permanent setups. [Citing](https://github.com/kubernetes-sigs/kind/issues/1867#issuecomment-698611610) kind author Benjamin Elder:
> We do not want users becoming highly attached to their kind clusters. Testing should be from a clean state and critical data should not be stored permanently in these clusters. They should start quickly and be disposable.

Kind uses random ports for each cluster, and configures kubectl contexts accordingly, which simplifies setting up several kube clusters running in parralel.

```text
$ kind create cluster --name kind1 --image rossgeorgiev/kind-node-arm64
...
$ kubectl config get-contexts
CURRENT   NAME         CLUSTER      AUTHINFO     NAMESPACE
          kind-kind    kind-kind    kind-kind    
*         kind-kind1   kind-kind1   kind-kind1   
```

## K3D + k3s

Install k3d:
```text
$ curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | TAG=v4.0.0 bash
```
