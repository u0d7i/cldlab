# Docker install

```text
$ sudo apt update

$ sudo apt install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common

$ curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -

$ echo "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
    $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list

$ sudo apt update

$ sudo apt install -y --no-install-recommends \
    docker-ce \
    cgroupfs-mount

$ sudo usermod -aG docker user

$ newgrp docker

$ docker info
...
WARNING: No memory limit support
WARNING: No swap limit support
WARNING: No kernel memory TCP limit support
WARNING: No oom kill disable support
...
```

Fix cgroup memory:

Add `cgroup_memory=1 cgroup_enable=memory swapaccount=1` to `/boot/cmdline.txt`
```
$ sudo vi /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=f76ca8fb-02 rootfstype=ext4 elevator=deadline fsck.repair=yes cgroup_memory=1 cgroup_enable=memory swapaccount=1 rootwait

$ sudo reboot
```

Verify:

```
$ docker info

$ docker version

$ docker run --rm hello-world
```

## Change Docker data dir:

- stop docker service:
```text
$ sudo service docker stop
```
- create docker daemon config:
```text
$ echo '{ "data-root": "/media/data/docker" }' | sudo tee /etc/docker/daemon.json
```
- create directory:
```text
$ sudo mkdir -p /media/data/docker
```
- sync data structure:
```text
$ sudo rsync -axPS /var/lib/docker/ /media/data/docker
```
- start docked service:
```text
$ sudo service docker start
```
- verify data dir:
```text
$ docker info | grep 'Docker Root Dir'
```
