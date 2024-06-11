---
title: Hyper-V
weight: 2
pre: "<b></b>"
chapter: true
---

## Setting Up Ubuntu VMs on Windows using Hyper-V

In this section, you will learn how to create two Ubuntu virtual machines (VMs) on your Windows machine using Hyper-V, configure a virtual switch to allow the VMs to connect to your physical network, and install containerd and related components on the Ubuntu VMs.

### Prerequisites

- Windows 10 Pro, Enterprise, or Education with Hyper-V enabled
- Ubuntu Server ISO image (download from [Ubuntu](https://ubuntu.com/download/server))

### Step 1: Enable Hyper-V on Windows

1. Open "Control Panel" and navigate to "Programs" -> "Turn Windows features on or off".
2. Check the box for "Hyper-V" and click "OK".
3. Restart your computer if prompted.

### Step 2: Create a Virtual Switch

1. Open "Hyper-V Manager".
2. In the right pane, click on "Virtual Switch Manager".
3. Select "New virtual network switch" and choose "External" as the type.
4. Click "Create Virtual Switch".
5. Give the switch a name (e.g., "ExternalSwitch").
6. Under "Connection type", select the network adapter that connects to your physical network.
7. Click "OK" to create the virtual switch.

### Step 3: Create Ubuntu Virtual Machines

#### Creating the First VM

1. In "Hyper-V Manager", click on "New" -> "Virtual Machine".
2. Follow the wizard to configure the VM:
   - Name: Ubuntu-1
   - Generation: Generation 1
   - Memory: Allocate at least 2GB
   - Network: Connect to the "ExternalSwitch" created earlier
   - Virtual Hard Disk: Create a new virtual hard disk (20GB recommended)
   - Installation Options: Point to the Ubuntu Server ISO image

3. Complete the wizard and start the VM.
4. Follow the Ubuntu installation steps to install Ubuntu Server on the VM.

## Configure IP

When configuring an Ubuntu virtual machine (VM) on Hyper-V to act as a Kubernetes node, it is important to allocate three virtual switches to ensure proper network segmentation and communication. Here’s a step-by-step guide to achieve this:

### 1. External Virtual Switch

The external virtual switch is used to bind the Ubuntu VM to the physical router of the Windows host machine. This requires manual configuration of a static IP address and setting up the default route in the `netplan` configuration.

#### Steps:

1. **Create External Virtual Switch**:
   - Open Hyper-V Manager.
   - Go to `Virtual Switch Manager`.
   - Select `New virtual network switch` and choose `External`.
   - Bind this switch to the network adapter connected to your physical router.

2. **Assign Static IP in Ubuntu**:
   - Edit the `netplan` configuration file, typically located at `/etc/netplan/01-netcfg.yaml` or a similar path.
   - Configure the static IP and set the default route to use this external interface. Example configuration:

     ```yaml
         ...
            eth0:
                  addresses:
                  - 192.168.1.100/24
                  routes:
                  - to: default
                  via: 192.168.1.1
                  nameservers:
                     addresses: []
                     search: []
         ...
     ```

3. **Apply the Netplan Configuration**:
   ```sh
   sudo netplan apply
   ```

### 2. Internal Network for VM-to-Host Communication
The second switch establishes a local network between the VM and the Windows host machine. This allows for clear distinction of VMs on different physical machines, facilitating horizontal scaling.

#### Steps:

1. **Create Internal Virtual Switch**:
   - Open Hyper-V Manager.
   - Go to `Virtual Switch Manager`.
   - Select `New virtual network` switch and choose `Internal`.
   - Assign IP Addresses:


Assign an IP address to the Windows host for this network, e.g., 10.0.1.1 for Windows 1 and 10.0.2.1 for Windows 2.

Configure the Ubuntu VM to use an IP address in the same subnet. Example netplan configuration:

2. **Assign Static IP in Ubuntu**:
   - Edit the `netplan` configuration file, typically located at `/etc/netplan/01-netcfg.yaml` or a similar path.
   - Configure the static IP and set the default route to use this external interface. Example configuration:

```yaml
      ...
         eth1:
               addresses:
               - 10.0.1.100/24
               nameservers:
                  addresses: []
                  search: []
      ...
```
3. **Apply the Netplan Configuration**:
   ```sh
   sudo netplan apply
   ```

#### Configure Windows Host IP Address:

- Go to `Control Panel` -> `Network and Internet` -> `Network and Sharing Center`.
- Click on Change adapter settings.
- Right-click on the internal virtual switch and select `Properties`.
- Select `Internet Protocol Version 4 (TCP/IPv4)` and click `Properties`.
- Set the IP address to `10.0.1.1`  and the subnet mask to `255.255.255.0`.


Open a cmd or powershell window, the try connect the ubuntu with

```sh
ssh <your-ubuntu-usernane>@10.0.1.100
```

and go back to ubuntu vm try to ping windows host 

```sh
ping 10.0.1.1
```

### 2. Default Switch for External Connectivity (Optional)

Ensure the VM has internet access by assigning a default switch or ensuring the external virtual switch provides internet connectivity.

#### Steps:

1. Assign Default Switch (Optional but Recommended):

   - Open Hyper-V Manager.
   - Go to `Virtual Switch Manager`.
   - Select `New virtual network` switch and choose `Default Switch`.
   - Configure External Connectivity:

If using the external virtual switch for internet access, ensure it can reach external networks like the Google Kubernetes repository. Example netplan configuration:

2. **Assign Static IP in Ubuntu**:
   - Edit the `netplan` configuration file, typically located at `/etc/netplan/01-netcfg.yaml` or a similar path.
   - Configure the static IP and set the default route to use this external interface. Example configuration:

```yaml
   ...
    eth2:
      dhcp4: true
   ...
```        
Apply the Netplan Configuration:

3. **Apply the Netplan Configuration**:
   ```sh
   sudo netplan apply
   ```

Ping an external site like Google to ensure internet connectivity:

```sh
ping google.com
```

By following these steps, you can effectively set up an Ubuntu VM on Hyper-V as a Kubernetes node with proper network configuration, ensuring it is well-integrated with both the Windows host and the external network.

### Finnal Configuration File Example

```yaml
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            addresses:
            - 192.168.1.100/24
            routes:
            - to: default
              via: 192.168.1.1
            nameservers:
                addresses: []
                search: []
        eth1:
            addresses:
            - 10.0.1.100/24
            nameservers:
                addresses: []
                search: []
        eth2:
           dhcp4: true
    version: 2
```
