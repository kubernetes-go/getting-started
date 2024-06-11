---
title: Intsall kubeadm/kubectl/kubelet
weight: 1
pre: "<b></b>"
chapter: true
---

# Setting Up Kubernetes Nodes with kubeadm, kubectl, and kubelet

After configuring the network for your Ubuntu VMs on Hyper-V, the next step is to install the necessary Kubernetes components (`kubeadm`, `kubectl`, and `kubelet`) on all nodes, including both the control plane and worker nodes. Ensure that the default route is configured correctly so that all nodes can communicate with each other through the appropriate switch or router.

## 1. Install kubeadm, kubectl, and kubelet

### Steps:

1. **Update the package list**:

   ```sh
   sudo apt-get update
   ```
2. **Install kubelet/kubeadm/kubectl**
   ```sh
    sudo apt-get update
    # apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg

    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
   ```