---
title: cotainerd
weight: 3
pre: "<b></b>"
chapter: true
---

## Installing Containerd, runc, and CNI on Ubuntu VMs for Kubernetes Nodes

> More details can be found form https://github.com/containerd/containerd/blob/main/docs/getting-started.md

In this guide, you will learn how to install `containerd`, `runc`, and `CNI` plugins on Ubuntu virtual machines, and verify the setup by pulling and running a Redis image. Additionally, you will install `nerdctl` to manage images and containers.

### Step 1: Update System Packages

First, update the system packages and install necessary dependencies:

```sh
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common
```

### Step 2: Install Containerd
Install containerd:

```sh
sudo apt install -y containerd
```

## Configure containerd:

Create the default configuration file:

```sh
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

edit the configuration file:

```sh
vi /etc/containerd/config.toml
```

set `SystemdCgroup` to `true` in the `plugins.cri.containerd.runtimes.runc.options` section:

```yaml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```


### Restart and enable the containerd service:

```sh
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Step 3: Install runc
Download and install runc:

```sh
sudo apt install -y runc
```

### Step 4: Install CNI Plugins

Download and install CNI plugins:

```sh
sudo mkdir -p /opt/cni/bin
curl -L https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz | sudo tar -C /opt/cni/bin -xz
```

### Step 5: Verify Containerd Setup
Use ctr (the CLI tool for containerd) to pull the Redis image:

```sh
sudo ctr image pull docker.io/library/redis:latest
```

Check if the Redis image was successfully pulled:

```sh
sudo ctr image ls | grep redis
```

### Step 6: Install Nerdctl

`nerdctl` is a Docker-compatible CLI for managing containerd containers.

Download and install nerdctl:

```sh
sudo curl -L https://github.com/containerd/nerdctl/releases/download/v1.2.0/nerdctl-1.2.0-linux-amd64.tar.gz | sudo tar -C /usr/local/bin -xz
```

Verify that nerdctl was installed successfully:

```sh
nerdctl --version
```

### Step 7: Verify Nerdctl Setup
Use nerdctl to pull the Redis image (optional, skip if already pulled using ctr):

```sh
sudo nerdctl pull redis:latest
```

Check if the Redis image was successfully pulled:

```sh
sudo nerdctl images | grep redis
```
Use nerdctl to run a Redis container:

```sh
sudo nerdctl run -d --name redis-server -p 6379:6379 redis:latest
```
Check if the container is running:

```sh
sudo nerdctl ps
```
Verify that the Redis service is accessible:

```sh
redis-cli -h 127.0.0.1 -p 6379 ping
```

Expected output:

```
PONG
```

Ensure All Nodes Are Properly Configured
Repeat the above steps on each Kubernetes node to ensure they can all properly use images to start services.

## Conclusion
By following these steps, you have successfully installed containerd and its related components (runc and CNI) on your Ubuntu VMs. You verified the setup by pulling and running a Redis image, and installed nerdctl to manage images and containers. These configurations and validations ensure that each node can properly use images to start services, laying the groundwork for your Kubernetes cluster deployment.