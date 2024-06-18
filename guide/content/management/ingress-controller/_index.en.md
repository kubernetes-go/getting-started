---
title: Ingress Controller
weight: 2
chapter: true
pre: "<b></b>"
---

# Install Traefik as Ingress Controller

## Prerequisites

- A running Kubernetes cluster with MetalLB configured for external IP allocation.
- `kubectl` command-line tool configured to interact with your cluster.
- Helm installed in your cluster, if not, please do as [USING HELM FOR LOCAL CONTINUOUS DELIVERY WITH KUBERNETES]({{% ref "management/ci-cd/helm.md" %}}).
- Access to modify the hosts file on your host machine.
and run the following cmd



## Step 1: Install Traefik Using Helm

```sh
helm repo add traefik https://traefik.github.io/charts
helm repo update
kubectl create ns traefik-v2
# Install in the namespace "traefik-v2"
helm install --namespace=traefik-v2 \
    traefik traefik/traefik
```

## Step 2: Deploy the traefik/whoami Sample Service


**Create a new deployment for the traefik/whoami service**.


```yaml
cat <<EOF | kubectl apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami
spec:
  replicas: 1
  selector:
    matchLabels:
      app: whoami
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
      - name: whoami
        image: traefik/whoami
        ports:
        - containerPort: 80
EOF
```

**Expose the Deployment as a Service**

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: whoami
spec:
  selector:
    app: whoami
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
EOF
```

**Create an IngressRoute**

Create an IngressRoute to route traffic to the `whoami` service.


```yaml
cat <<EOF | kubectl apply -f -
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingressroute
  namespace: default

spec:
  entryPoints:
    - web

  routes:
  - match: Host(`www.foo-bar-local.com`)
    kind: Rule
    services:
    - name: whoami
      port: 80
EOF
```

## Step 3: Check the External IP

Verify that the whoami service has been assigned an external IP by land balancer.

```sh
kubectl get svc whoami
```

The output should include an `EXTERNAL-IP` column showing the assigned IP address.

## Step 4: Modify the Hosts File on Your Host Machine

Add an entry to your host machine's hosts file to map `www.foo-bar-local.com` to the external IP assigned to the whoami service.

### On macOS/Linux
Edit the /etc/hosts file:

```sh
sudo nano /etc/hosts
```

Add the following line:

```sh
<EXTERNAL-IP> www.foo-bar-local.com
```

### On Windows

Edit the `C:\Windows\System32\drivers\etc\hosts` file:

```bash
notepad C:\Windows\System32\drivers\etc\hosts
```
Add the following line:

```sh
<EXTERNAL-IP> www.foo-bar-local.com
```

## Step 5: Access the whoami Service

Open a web browser on your host machine and navigate to:

```sh
http://www.foo-bar-local.com
```

## Step 6: Automatically Obtain Let's Encrypt Certificates with Traefik

### Prerequisites
- A registered domain name with Cloudflare.
- A configured StorageClass in your Kubernetes cluster (e.g., NFS) for persistent storage required by Traefik to store Let's Encrypt certificates, you can install [Network File System(NFS) Storage Class for Kubernetes]({{% ref "management/storage/nfs.md" %}}).
- A Cloudflare API token with permissions to manage DNS, more detail can be found from [developers.cloudflare.com/create-token](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/)


### Save Cloudflare API Token to Secret

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare
  namespace: traefik-v2
type: Opaque
stringData:
  token: <your-cloudflare-api-token-here>
EOF
```

### Update the Traefik Helm Configuration

Create a Traefik values override file `values.traefik.yaml` with the following content:

```yaml
env:
- name: CF_DNS_API_TOKEN
  valueFrom:
    secretKeyRef:
      name: cloudflare
      key: token
persistence:
  # -- Enable persistence using Persistent Volume Claims
  # ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
  # It can be used to store TLS certificates, see `storage` in certResolvers
  enabled: true
  name: data
  #  existingClaim: ""
  accessMode: ReadWriteMany
  size: 128Mi
  storageClass: nfs-client
  # volumeName: "nfs-traefik-certResolvers"
  path: /data
  annotations: {}
  # -- Only mount a subpath of the Volume into the pod
  # subPath: ""

# -- Certificates resolvers configuration.
# Ref: https://doc.traefik.io/traefik/https/acme/#certificate-resolvers
# See EXAMPLES.md for more details.
certResolvers: 
  letsencrypt:
    email: "<YOUR_EMAIL>"
    dnsChallenge:
      provider: cloudflare
    storage: /data/acme.json
```
Replace <YOUR_EMAIL> with your email address.

**Apply the Updated Configuration**
Upgrade your Traefik installation with the new configuration.
```sh
helm upgrade --install traefik traefik/traefik -f values.traefik.yaml --namespace=traefik-v2
```

### Update the IngressRoute for TLS
Modify your IngressRoute to use the Let's Encrypt resolver:

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingressroute
  namespace: default
spec:
  entryPoints:
    - web
    - websecure
  routes:
  - match: Host(`www.foo-bar-local.com`)
    kind: Rule
    services:
    - name: whoami
      port: 80
  tls:
    certResolver: letsencrypt
EOF
```

### Access the Secure Service
Open a web browser on your host machine and navigate to:

```sh
https://www.foo-bar-local.com
```

Your browser should display a secure connection with a Let's Encrypt certificate.

## Force rediect to HTTPS from HTTP

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: redirect-http-to-https
spec:
  redirectScheme:
    scheme: https
    permanent: true
```

and use it for your ingress route

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingressroute-http
  namespace: default

spec:
  entryPoints:
    - web

  routes:
  - match: Host(`www.foo-bar-local.com`)
    middlewares:
      - name: redirect-http-to-https
    kind: Rule
    services:
    - name: whoami
      port: 80
---

apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: whoami-ingressroute-https
  namespace: default

spec:
  entryPoints:
    - websecure

  routes:
  - match: Host(`www.foo-bar-local.com`)
    kind: Rule
    services:
    - name: whoami
      port: 80
  tls:
    certResolver: letsencrypt
```
It will redirect to https from http.



## Access Traefik Dashboard


```sh
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```

then open url `http://127.0.0.1:9000/dashboard/#/` in browser.


### Expose Dashboard via Ingress Route

update your `values.traefik.yaml`

```yaml
additionalArguments: 
  - "--serversTransport.insecureSkipVerify=true" 
  - "--api.insecure=true" 
  - "--api.dashboard=true"

ports:
  traefik:
    expose:
      default: true

ingressRoute:
  dashboard:
    # -- Create an IngressRoute for the dashboard
    enabled: false
```


```sh
helm upgrade --install traefik traefik/traefik -f values.traefik.yaml --namespace=traefik-v2
```

then apply the ingress route 

```yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: traefik-dashboard-ingress-route
  namespace: traefik-v2
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`traefik.foo-bar-local.com`) && PathPrefix(`/dashboard`)
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
    - match: Host(`traefik.foo-bar-local.com`) && PathPrefix(`/api`)
      kind: Rule
      services:
        - name: api@internal
          kind: TraefikService
  tls:
    certResolver: letsencrypt
```