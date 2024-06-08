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

#### Creating the Second VM

Repeat the above steps to create the second VM, naming it "Ubuntu-2" and using the same configuration.

### Step 4: Configure Network Settings

1. After both VMs are installed, ensure they are both connected to the "ExternalSwitch".
2. Verify that the VMs can reach the internet and each other by checking their IP addresses and using `ping` commands.