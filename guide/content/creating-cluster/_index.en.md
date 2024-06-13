---
title: Creating Cluster
weight: 3
chapter: true
pre: "<b>3. </b>"
---

# Initializing the Kubernetes Control Plane with kubeadm
To set up the Kubernetes control plane, follow these steps:

## Initialize the Control Plane
Run the following command on the control plane node. Replace 192.168.1.100 with the IP address of your control plane node:

    ```sh
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.1.100
    ```


Set Up Local kubeconfig
After the control plane is initialized, set up the local kubeconfig file to interact with the cluster:

    ```sh
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

Install Flannel as the network plugin:

```sh
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Joining Worker Nodes to the Cluster
To join worker nodes to the cluster, follow these steps on each worker node.

1. Run the Join Command
On each worker node, run the kubeadm join command provided by the kubeadm init output. This command looks something like this:

```sh
sudo kubeadm join 192.168.1.100:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

2. Verify Node Joining
After joining the nodes, verify that they have successfully joined the cluster:

```sh
kubectl get nodes
```

You should see the worker nodes listed along with the control plane node.

## Try to deploy a demo application

You have successfully initialized a Kubernetes control plane using kubeadm, joined worker nodes to the cluster, and deployed a simple HTTP Echo application using Helm. This setup provides a foundation for further Kubernetes exploration and application deployment.


### Deploying Nginx to Test Worker Nodes
Now, let's deploy an Nginx application to ensure that the worker nodes can pull images and run pods correctly.

1. Create Nginx Deployment
Use the following command to create an Nginx deployment:

```sh
kubectl create deploy nginx --image nginx:latest
```

2. Verify Nginx Deployment
Check the status of the Nginx deployment to ensure it was successful:

```sh
kubectl get pods
```
You should see a pod with the name nginx running. To get more details about the pod, use:

```sh
kubectl describe pod <pod-name>
```

3. Expose Nginx Service
Expose the Nginx deployment as a service to get a Cluster IP:

```sh
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
```

4. Get Nginx Service IP
Get the Cluster IP assigned to the Nginx service:

```sh
kubectl get svc nginx
```

5. Test Nginx Service
Test if the Nginx service is accessible by pinging the Cluster IP from within the cluster. Use a temporary pod for this:

```sh
kubectl run tmp-shell --rm -i --tty --image busybox -- /bin/sh
```

In the temporary shell, use the following commands:

```sh
wget -qO- http://<Cluster-IP>
```

Replace <Cluster-IP> with the actual Cluster IP of the Nginx service.

If everything is set up correctly, you should see the default Nginx welcome page content.

Finally, please remember to delete your demo deployment and service.

```sh
kubectl delete svc nginx
kubectl delete deployment nginx
```

## Install Load Balancer 

```sh
kubectl edit configmap -n kube-system kube-proxy
```

and set:

```yaml
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"
ipvs:
  strictARP: true
```

To install MetalLB, apply the manifest:

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.5/config/manifests/metallb-native.yaml
```

then create a file named `metalLb-ip.yaml`

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.120-192.168.1.160
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
```

try to apply it

```sh
kubectl apply -f metalLb-ip.yaml
```

### Verify Extenal IP
try to verify the external ip for a service with type LoadBalancer

```sh
kubectl create deploy nginx --image nginx:latest
kubectl expose deploy nginx --port 80 --type LoadBalancer
```

Check the service to get the external IP assigned by MetalLB.
```sh
kubectl get svc nginx
```

The output should look something like this:
```
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
nginx   LoadBalancer   10.96.222.240    192.168.1.120   80:30091/TCP   1m
```
then you can test the nginx via external ip


With the external IP assigned (e.g., 192.168.1.120), you can now test access to the Nginx service from your host machine or any device in the same network.

Open a web browser and navigate to: 

```sh
http://192.168.5.200
```

You should see the default Nginx welcome page.

or just run a simple cmd

```sh
curl http://192.168.5.200
```sh


if successed, we can delete the nginx

```sh
kubectl delete deploy nginx
kubectl delete svc nginx
```