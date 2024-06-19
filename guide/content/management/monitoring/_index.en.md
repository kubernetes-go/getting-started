---
title: Monitoring
weight: 4
chapter: true
pre: "<b></b>"
---

# Node Metrics

## Install Metrics Server 

```sh
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Kubelet certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing --kubelet-insecure-tls to Metrics Server)

```yaml
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls # add this line
```

### Check resources

```sh
kubectl top nodes
```

{{% children  %}}