---
title: Ubuntu
weight: 5
pre: "<b></b>"
chapter: true
---

## Setting Up Ubuntu VMs on Ubuntu Host using VirtualBox

In this section, you will learn how to create two Ubuntu virtual machines (VMs) on your Ubuntu host using VirtualBox, configure networking so that the VMs can connect to your physical network, and install containerd and related components on the Ubuntu VMs.

### Prerequisites

- Ubuntu host machine with internet access
- VirtualBox installed on the Ubuntu host
- Ubuntu Server ISO image (download from [Ubuntu](https://ubuntu.com/download/server))

### Step 1: Install VirtualBox

> More information on installing VirtualBox can be found [here](https://www.virtualbox.org/wiki/Linux_Downloads).

1. Open a terminal on your Ubuntu host.
2. Add the Oracle VirtualBox repository:

    ```bash
    echo "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
    ```

3. Download and add the Oracle public key for the repository:

    ```bash
    wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
    wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
    ```

4. Update the package list and install VirtualBox:

    ```bash
    sudo apt update
    sudo apt install virtualbox-6.1
    ```

### Step 2: Download Ubuntu Server ISO

1. Download the Ubuntu Server ISO image from the [Ubuntu website](https://ubuntu.com/download/server).
2. Save the ISO file to a known location on your Ubuntu host.

### Step 3: Create Ubuntu Virtual Machines

#### Creating the First VM

1. Open VirtualBox from the application menu.
2. Click on "New" to create a new virtual machine.
3. Follow the wizard to configure the VM:
   - Name: Ubuntu-1
   - Type: Linux
   - Version: Ubuntu (64-bit)
   - Memory: Allocate at least 2GB
   - Hard disk: Create a virtual hard disk now (20GB recommended)
   - Hard disk file type: VDI (VirtualBox Disk Image)
   - Storage on physical hard disk: Dynamically allocated

4. Once the VM is created, select it and click on "Settings".
5. Under "System" -> "Motherboard", ensure that "Enable EFI" is unchecked.
6. Under "Storage", click on the "Empty" optical drive and then click on the disk icon to choose a disk file. Select the Ubuntu Server ISO you downloaded earlier.
7. Under "Network", set "Attached to" to "Bridged Adapter" and select your physical network adapter.

8. Click "OK" to save the settings and start the VM.
9. Follow the Ubuntu installation steps to install Ubuntu Server on the VM.

#### Creating the Second VM

Repeat the above steps to create the second VM, naming it "Ubuntu-2" and using the same configuration.

### Step 4: Configure Network Settings

1. After both VMs are installed, ensure they are both connected to the "Bridged Adapter".
2. Verify that the VMs can reach the internet and each other by checking their IP addresses and using `ping` commands.
