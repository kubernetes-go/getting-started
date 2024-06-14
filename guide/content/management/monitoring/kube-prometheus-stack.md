---
title: Kube-Prometheus-Stack
weight: 1
chapter: true
pre: "<b></b>"
---

# Guide to Installing and Using Kube-Prometheus-Stack in Kubernetes

This guide will walk you through installing the Kube-Prometheus-Stack using Helm in a Kubernetes cluster. We'll create a new namespace, access Grafana's web UI via port forwarding, add Prometheus as a data source, explore important dashboards, and expose Grafana through an Ingress controller.

## Prerequisites
- A running Kubernetes cluster
- Helm installed on your local machine
- kubectl configured to interact with your cluster


## Install with Helm

### Step 1: Create a Namespace
First, create a new namespace called monitoring to keep all monitoring-related resources isolated.

```sh
kubectl create namespace monitoring
```

### Step 2: Add Helm Repository and Install Kube-Prometheus-Stack
Add the Helm repository and install the Kube-Prometheus-Stack chart.

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring
```

### Step 3: Access Grafana's Web UI via Port Forwarding
Once the stack is installed, you can access Grafana's web UI by port forwarding.

```sh
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```
Open your web browser and navigate to http://localhost:3000.

**Grafana Login**
The default login credentials are:

- Username: admin
- Password: prom-operator
You can retrieve the password using:

```sh
kubectl get secret -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Step 4: Add Prometheus as a Data Source in Grafana

- Log in to Grafana.
- Click on the "Gear" icon (Configuration) on the left sidebar.
- Select "Data Sources".
- Click "Add data source".
- Select "Prometheus".
- Set the URL to http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090.
- Click "Save & Test".

### Step 5: Explore Default Dashboards
Grafana comes with several pre-configured dashboards. Some important ones to check are:

- Kubernetes / Compute Resources / Node (Pods): Shows CPU and memory usage for each node.
- Kubernetes / Compute Resources / Namespace (Pods): Shows resource usage per namespace.
- Kubernetes / Compute Resources / Pod: Shows resource usage per pod.

To access these dashboards:

- Click on the "Dashboard" icon on the left sidebar.
- Select "Manage".
- Browse through the folders and select the dashboard you want to view.

### Step 6: Expose Grafana using an Ingress Controller
If you have an Ingress controller installed (like Nginx, Traefik, etc.), you can expose Grafana externally.

Create an Ingress resource for Grafana. Save the following YAML into a file named grafana-ingress.yaml.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: grafana.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kube-prometheus-stack-grafana
            port:
              number: 80
  tls:
  - hosts:
    - grafana.yourdomain.com
    secretName: grafana-tls
```

Apply the Ingress resource.

```sh
kubectl apply -f grafana-ingress.yaml
```

Make sure to replace `grafana.yourdomain.com` with your actual domain name and set up the necessary DNS records. If you have TLS/SSL certificates, replace `grafana-tls` with your actual secret name containing the certificates.

### Trouble Shooting

If Grafana can not connect to Promethues datasouce, try to check the internal network connnect

```sh
kubectl.exe 'exec' '-it' 'kube-prometheus-stack-grafana-544d547ffd-77q4f' '--namespace' 'monitoring' '--container' 'grafana' '--' 'bash'
```

try to run `curl` to check the network:

```sh
curl http://kube-prometheus-stack-prometheus.monitoring:9090/
```
if you see the following error

```
curl: (6) Could not resolve host: kube-prometheus-stack-prometheus.monitoring
```
that means the `core dns` is not working as we expected.
and we can add more parameter with `nslook`

```sh
nslookup -debug kube-prometheus-stack-prometheus.monitoring 
```
then if you see

```
Server:         10.96.0.10
Address:        10.96.0.10:53

Query #0 completed in 40ms:
** server can't find kube-prometheus-stack-prometheus.monitoring: NXDOMAIN

Query #1 completed in 175ms:
** server can't find kube-prometheus-stack-prometheus.monitoring: NXDOMAIN
```

so let restart the `core dns`

```sh
kubectl -n kube-system rollout restart deployment coredns
```



