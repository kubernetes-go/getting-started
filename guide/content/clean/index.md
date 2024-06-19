---
title: Uninstall and Remove
weight: 6
chapter: true
pre: "<b>6. </b>"
---

## Step 1: Remove the Node from the Cluster

1. Mark the Node as Unschedulable

    ```bash
    kubectl cordon kube-worker-3
    ```

2. Drain the Node:

    ```bash
    kubectl drain kube-worker-3 --ignore-daemonsets --delete-local-data
    ```
3. Delete the Node:

    Remove the node from the Kubernetes cluster:

    ```bash
    kubectl delete node kube-worker-3  
    ```
## Step 2: Clean Up Flannel and CNI Configurations on the Node

Execute the following steps on the node to clean up Flannel and CNI configurations:

1. Stop and Disable kubelet and containerd Services:


    ```bash
    sudo kubeadm reset
    sudo systemctl stop kubelet
    sudo systemctl stop containerd
    sudo systemctl disable kubelet
    sudo systemctl disable containerd
    sudo apt-get remove kubelet kubeadm kubectl
    ```

2. Remove Flannel Configurations and Data:

    ```bash
    sudo rm -rf /var/lib/cni/
    sudo rm -rf /var/lib/kubelet/*
    sudo rm -rf /etc/cni/
    sudo rm -rf /run/flannel/
    ```

3. Remove CNI Configurations and Data:

    ```bash
    sudo rm -rf /etc/cni/net.d/
    sudo rm -rf /var/lib/cni/
    # sudo rm -rf /opt/cni/bin/
    ```

4. Remove containerd Data:

    ```bash
    sudo rm -rf /var/lib/containerd/
    sudo rm -rf /etc/kubernetes/kubelet.conf 
    sudo rm -rf /etc/kubernetes/pki/ca.crt
    # sudo rm -rf /etc/systemd/system/containerd.service.d/http-proxy.conf
    # sudo rm -rf /etc/kubernetes/bootstrap-kubelet.conf
    ```

5. Reboot the Node:

    ```bash
    sudo reboot
    ```
6. Verify the Cleanup

   Use the following command to verify that the node has been removed from the Kubernetes cluster:

    ```bash
    kubectl get nodes
    ```

7. Verify the Cleanup is Complete:

    ```bash
    sudo ls /var/lib/kubelet
    sudo ls /etc/cni/net.d/
    sudo ls /opt/cni/bin/
    sudo ls /var/lib/containerd/
    ```

If these directories do not exist or are empty, the cleanup is complete. 

## Reset Control Plane

```sh
kubeadm reset
sudo rm -rf /etc/cni/net.d/
sudo rm -rf /var/lib/cni/
sudo rm -rf $HOME/.kube/config
sudo reboot
```