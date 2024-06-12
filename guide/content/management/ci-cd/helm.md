---
title: Helm
weight: 3
chapter: true
pre: "<b></b>"
---

# Using Helm for Local Continuous Delivery with Kubernetes

## Installing Helm

Helm is a package manager for Kubernetes that helps you manage Kubernetes applications. The following steps will guide you through installing Helm locally.

### 1. Download and Install Helm

First, download the Helm executable for your operating system. You can get the latest version from the [official Helm GitHub repository](https://github.com/helm/helm/releases).

For macOS users, you can install Helm using Homebrew:

```sh
brew install helm
```

For Windows users, you can install Helm using Chocolatey:

```sh
choco install kubernetes-helm
```

For Linux users, you can use the following command:

```sh
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### 2. Ensuring kube/config File Validity
After installation, verify that Helm is installed correctly:

```sh
helm version
```

Before using Helm, ensure that your local kube/config file is valid and that you can connect to your Kubernetes control plane node. This is crucial for Helm to interact properly with your Kubernetes cluster.

1. Verify kube/config File
Check the current Kubernetes context to make sure you are connected to the correct cluster:

    ```sh
    kubectl config current-context
    ```

2. Test Cluster Connectivity
Test the connectivity to the Kubernetes control plane by listing the nodes in the cluster:

    ```sh
    kubectl get nodes
    ```
If you see a list of nodes, your configuration is correct, and you are connected to the cluster. If there are any issues, you might need to set the correct context or update your kube/config file.