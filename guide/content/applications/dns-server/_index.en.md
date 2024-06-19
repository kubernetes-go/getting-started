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

apply the following ingress route if you already installed [TRAEFIK AS INGRESS CONTROLLER]({{% ref "management/ingress-controller" %}}).

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

## Set Pi-Hole as Upstream Dns Server to CoreDns

get the external IP of `pihole-dns-tcp`

```sh
kubectl get svc -n default pihole-dns-tcp
NAME             TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
pihole-dns-tcp   LoadBalancer   10.105.21.63   192.168.5.202   53:30891/TCP   2d1h
```

edit your `values.pi-hole.yaml`, add these

```yaml
dnsmasq:
  customDnsEntries:
    - address=/foo-bar-local.com/192.168.5.202
```

then update your helm release 

```sh
helm upgrade --install pihole mojo2600/pihole -f values.pi-hole.yaml
```

### Add config for CoreDns

```sh
vi coredns-custom.yaml
```

edit it to 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  custom.server: |
    foo-bar-local.com:53 {
       errors
       cache 30
       forward . 192.168.5.202
    }
```
  {{% notice style="tip" %}}
this means all requestes to domain `foo-bar-local.com` will query the dns record from server `192.168.5.202`. `foo-bar-local.com` can be replaced bu your customerize domian.
  {{% /notice %}}

then try to monitor the logs 

```sh
kubectl logs -n kube-system -l k8s-app=kube-dns
```

if coredns pods does not restart, please try to triiger manually

```sh
kubectl rollout restart deployment coredns -n kube-system
```


### Customize your domain in cluster

  {{% notice style="note" %}}
Please install `whoami` service in `default` namespace, more details please go to [Install Traefik Using Helm]({{% ref "management/ingress-controller" %}}).
  {{% /notice %}}


you can access your service in cluster by these names 

- `service-name.namespace.svc.cluster.local`
- `service-name.namespace`

install `netshoot`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: netshoot
spec:
  containers:
    - name: netshoot
      image: nicolaka/netshoot
      command: ["sleep", "3600"]
```

### Ensure internal dns works

run shell in pod

```sh
kubectl 'exec' '-it' 'netshoot' '--namespace' 'default' '--container' 'netshoot' '--' 'sh'
```

```sh
traceroute whoami.default
traceroute to whoami.default (10.107.156.209), 30 hops max, 46 byte packets
 1  whoami.default.svc.cluster.local (10.107.156.209)  0.010 ms  0.008 ms  0.006 ms
```

### Try external dns

go to `pi-hole` admin portal, add a A record

```sh
whoami.foo-bar-local.com 192.168.5.202
```

then go back to `busybox`

```sh
traceroute whoami.foo-bar-local.com
```
