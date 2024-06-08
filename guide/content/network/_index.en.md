---
title: Configuring the network of Kubernetes Node
weight: 10
chapter: true
pre: "<b>2. </b>"
---

## Configuring Network Settings for Kubernetes Nodes

In this section, you will learn how to configure the network settings for your Ubuntu VMs to ensure they are on the same local network segment. This is necessary for both control plane and worker nodes. We will set a default route through the shared router and assign static IP addresses to each VM.

### Step 1: Verify Default Route

Ensure that each VM is connected to the same virtual switch that bridges to your host machine's network adapter. This setup should already be in place if you followed the previous steps to create the VMs with a "Bridged Adapter" network setting.

1. Open a terminal on each VM and verify the network configuration with the following command:

    ```bash
    ip route
    ```

    The output should include a default route that points to your physical network's router. For example:

    ```plaintext
    default via 192.168.10.1 dev eth0
    ```

2. If the default route is missing or incorrect, you may need to recheck your VirtualBox network settings to ensure the "Bridged Adapter" is properly configured.

### Step 2: Assign Static IP Addresses

To assign static IP addresses to each VM, edit the network configuration file. The configuration will vary slightly depending on whether you are using `netplan` (common in newer Ubuntu versions) or `ifupdown` (common in older versions).

#### Using Netplan (Ubuntu 18.04 and later)

1. Open the netplan configuration file in a text editor:

    ```bash
    sudo nano /etc/netplan/01-netcfg.yaml
    ```

2. Configure the file to set a static IP address. Below is an example configuration for the control plane node (192.168.10.100):

    ```yaml
    network:
      version: 2
      ethernets:
        eth0:
          addresses:
            - 192.168.10.100/24
          gateway4: 192.168.10.1
          nameservers:
            addresses:
              - 8.8.8.8
              - 8.8.4.4
    ```

3. Apply the netplan configuration:

    ```bash
    sudo netplan apply
    ```

Open a terminal on each node and run the following commands:

```bash
# Ping Google DNS to check internet connectivity
ping -c 4 8.8.8.8

# Ping Google's website
ping -c 4 google.com

# Ping Kubernetes official image repository
ping -c 4 k8s.gcr.io
```


### Next Step

Proceed to the next article:

### [Setting Up the Kubernetes Control Plane](#setting-up-the-kubernetes-control-plane)
