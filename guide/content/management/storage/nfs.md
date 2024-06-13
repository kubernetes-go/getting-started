---
title: Network File System(NFS)
weight: 1
chapter: true
pre: "<b></b>"
---

# Setting Up an NFS Server for Kubernetes

## Prerequisites

- A new Ubuntu virtual machine to act as the NFS server.
- This NFS server should be on the same network (subnet) as your Kubernetes nodes to ensure network accessibility.
- As this is for a local test environment, you don't need to worry about the physical disk type of the virtual hard drive. However, for production environments, it's recommended to use RAID-configured disks for important data storage, avoiding SSDs for database files like MySQL.

## Step 1: Configure the NFS Server

### Install NFS Server

First, install the NFS server package on the Ubuntu VM.

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```


### Create a Shared Directory
Create a directory to share via NFS.

```bash
sudo mkdir -p /srv/nfs/kubedata
sudo chown nobody:nogroup /srv/nfs/kubedata
sudo chmod 777 /srv/nfs/kubedata
```

### Configure NFS Exports
Edit the `/etc/exports` file to configure the shared directory.

```sh
sudo nano /etc/exports
```

Add the following line:

```bash
/srv/nfs/kubedata    *(rw,sync,no_subtree_check)
```

Save and close the file, then export the shared directory.

```bash
sudo exportfs -ra
```

### Start the NFS Server
Start and enable the NFS server.

```bash
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

## Step 2: Configure a Kubernetes Node to Use the NFS Server
Install NFS Common on Kubernetes Node
Choose one of your Kubernetes nodes and install the NFS common package.

```bash
sudo apt update
sudo apt install nfs-common -y
```

### Mount the NFS Share
Create a mount point and mount the NFS share to test connectivity.

```bash
sudo mkdir -p /mnt/nfs/kubedata
sudo mount -t nfs <nfs-server-ip>:/srv/nfs/kubedata /mnt/nfs/kubedata
```

Replace <nfs-server-ip> with the IP address of your NFS server.

### Test the NFS Mount
Create a test file to ensure the NFS mount is working correctly.

```bash
echo "NFS is working!" | sudo tee /mnt/nfs/kubedata/test.txt
cat /mnt/nfs/kubedata/test.txt
```

Check if the test.txt file is accessible from the NFS server and other nodes in the cluster.

## Step 3: Install NFS Subdir External Provisioner in the Kubernetes Cluster

### Install the Provisioner

- Add the Helm Repository
- Add the Helm repository for the NFS subdir external provisioner.

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --create-namespace \
  --namespace nfs-provisioner \
  --set nfs.server=<nfs-server-ip> \
  --set nfs.path=/srv/nfs/kubedata
```

Replace <nfs-server-ip> with the IP address of your NFS server.

## Step 4: Create a PVC and Use It in a Pod
Create a PVC
Create a PersistentVolumeClaim (PVC) using the NFS storage class.

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-test
  labels:
    storage.k8s.io/name: nfs
    storage.k8s.io/part-of: kubernetes-complete-reference
    storage.k8s.io/created-by: ssbostan
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  resources:
    requests:
      storage: 20Mi
EOF
```
### Deploy a Test Pod
Deploy a test pod using the PVC.

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: app
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/mnt/nfs"
          name: nfs-storage
  volumes:
    - name: nfs-storage
      persistentVolumeClaim:
        claimName: nfs-test
EOF
```

## Step 5: Verify the NFS Server
Check the NFS Server for the Pod Directory
On your NFS server, verify that the directory for the busybox pod has been created.

```bash
ls /srv/nfs/kubedata
```

You should see a directory corresponding to the PVC created by Kubernetes.

## Conclusion
By following these steps, you have successfully set up an NFS server on a new Ubuntu virtual machine and configured it to work with your Kubernetes cluster. You also installed the NFS subdir external provisioner, created a PVC, and verified that the NFS storage is accessible from your Kubernetes pods.

Important Details to Note
- Ensure the NFS server and all Kubernetes nodes are on the same network for smooth communication.
- For production environments, use RAID-configured disks for the NFS server to ensure data redundancy and reliability.
- Always verify network connectivity between the NFS server and Kubernetes nodes before proceeding with configurations.
- Regularly monitor your NFS server and perform backups to prevent data loss.

This setup helps in managing persistent storage needs for your Kubernetes cluster efficiently and allows you to focus on developing and deploying your applications.
