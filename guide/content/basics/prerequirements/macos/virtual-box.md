---
title: VirtualBox
weight: 2
pre: "<b></b>"
chapter: true
---
## Setting Up Ubuntu VMs on macOS using VirtualBox

In this section, you will learn how to create two Ubuntu virtual machines (VMs) on your macOS host using VirtualBox, configure networking so that the VMs can connect to your physical network, and install containerd and related components on the Ubuntu VMs.

### Prerequisites

- macOS host machine with internet access
- VirtualBox installed on the macOS host
- Ubuntu Server ISO image (download from [Ubuntu](https://ubuntu.com/download/server))

### Step 1: Install VirtualBox

1. Download VirtualBox for macOS from the [VirtualBox website](https://www.virtualbox.org/wiki/Downloads).
2. Open the downloaded `.dmg` file and follow the instructions to install VirtualBox on your macOS.

### Step 2: Download Ubuntu Server ISO

1. Download the Ubuntu Server ISO image from the [Ubuntu website](https://ubuntu.com/download/server).
2. Save the ISO file to a known location on your macOS host.

### Step 3: Create Ubuntu Virtual Machines

#### Creating the First VM

1. Open VirtualBox from the Applications folder.
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

### Step 5: Install containerd and Related Components on Ubuntu

#### Update and Upgrade Packages

Log in to each Ubuntu VM and run the following commands:

```bash
sudo apt update
sudo apt upgrade -y
