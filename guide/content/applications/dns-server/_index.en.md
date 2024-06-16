---
title: DNS Server
weight: 7
chapter: true
pre: "<b></b>"
---

# Install Pi-Hole

## Introduction
Pi-hole is a powerful and highly customizable DNS service that allows you to manage domain names within your network. It's especially useful in office environments where internal domain management is required. This guide will walk you through the steps to install and use Pi-hole on your Kubernetes cluster using Helm.

## Prerequisites
Before we begin, ensure you have the following:

- A running Kubernetes cluster.
- Helm installed on your local machine.
- kubectl configured to interact with your Kubernetes cluster.

## Step-by-Step Guide

### Step 1: Add the Helm Repository
First, you need to add the Helm repository for Pi-hole. Run the following command:

```sh
helm repo add mojo2600 https://mojo2600.github.io/pihole-kubernetes/
helm repo update
helm install pihole mojo2600/pihole
```

### Step 2: Access Pi-hole
To access the Pi-hole web interface, you need to forward the Pi-hole service port to your local machine. Run the following command:

```sh
kubectl port-forward svc/pihole 80:80
```

Open a web browser and navigate to `http://localhost/admin`. You should see the Pi-hole web interface where you can configure your DNS settings. the deault password is `admin`.

### Step 3: Configure Pi-hole

There are also a few configuration changes that need to be made to make the pi-hole work better.

create a file with name `values.pi-hole.yaml`

> `192.168.5.202` is the next ip that the load balance will assign.

```yaml
serviceWeb:
  loadBalancerIP: 192.168.5.202
  annotations:
    metallb.universe.tf/allow-shared-ip: pihole-svc
  type: LoadBalancer
serviceDns:
  loadBalancerIP: 192.168.5.202
  annotations:
    metallb.universe.tf/allow-shared-ip: pihole-svc
  type: LoadBalancer
persistentVolumeClaim:
  enabled: true
  storageClass: "nfs-client"
  accessModes:
    - ReadWriteMany
  subPath: "pihole"
  
adminPassword: "my!admin3password"
```

then update your helm release 

```sh
helm upgrade --install pihole mojo2600/pihole -f values.pi-hole.yaml
```

### Access pi-hole admin dashboard

`http://192.168.5.202/admin/login.php`, please replace the `http://192.168.5.202` with your load balancer ip, you can get the 

```sh
$ kubectl get svc pihole-web -o wide
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)                      AGE     SELECTOR
pihole-web   LoadBalancer   10.100.211.100   192.168.5.202   80:30127/TCP,443:30480/TCP   8m39s   app=pihole,release=pihole
```



## Add customize domain for you pi-hole admin portal

apply the following ingress route if you already installed [TRAEFIK AS INGRESS CONTROLLER]({{< ref "management/ingress-controller/_index.md" >}}).

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect-http-to-https
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: pi-hole-web-http
  namespace: default

spec:
  entryPoints:
    - web

  routes:
  - match: Host(`pi-hole.foo-bar-local.com`)
    middlewares:
      - name: redirect-http-to-https
    kind: Rule
    services:
    - name: pihole-web
      port: 80
---

apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: pi-hole-web-https
  namespace: default

spec:
  entryPoints:
    - websecure

  routes:
  - match: Host(`pi-hole.foo-bar-local.com`)
    kind: Rule
    services:
    - name: pihole-web
      port: 80
      
  tls:
    certResolver: letsencrypt
```