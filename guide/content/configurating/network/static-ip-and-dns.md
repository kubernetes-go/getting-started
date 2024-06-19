---
title: IP and DNS 
weight: 1
pre: "<b></b>"
chapter: true
---

##  Network - IP and DNS

In this section, you will learn how to configure the network settings for your Ubuntu VMs to ensure they are on the same local network segment. This is necessary for both control plane and worker nodes. We will set a default route through the shared router and assign static IP addresses to each VM.

### Step 1: Verify Default Route

Ensure that each VM is connected to the same virtual switch that bridges to your host machine's network adapter. This setup should already be in place if you followed the previous steps to create the VMs with a "Bridged Adapter" network setting.

1. Open a terminal on each VM and verify the network configuration with the following command:

```bash
ip route show
```

The output should include a default route that points to your physical network's router. For example:

```plaintext
default via 192.168.10.1 dev eth0
```

2. If the default route is missing or incorrect, you may need to recheck your VirtualBox network settings to ensure the "Bridged Adapter" is properly configured.

```sh
sudo ip route del default
sudo ip route add default via 192.168.10.1 dev eth0 proto static # 192.168.10.1 is the IP address of your router.
ip route show
```

### Step 2: Assign Static IP Addresses

To assign static IP addresses to each VM, edit the network configuration file. The configuration will vary slightly depending on whether you are using `netplan` (common in newer Ubuntu versions).

```sh
vi /etc/systemd/resolved.conf
```

```conf
[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
DNS=8.8.8.8
#FallbackDNS=
#Domains=
#DNSSEC=no
#DNSOverTLS=no
#MulticastDNS=no
#LLMNR=no
#Cache=no-negative
#CacheFromLocalhost=no
#DNSStubListener=yes
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
#StaleRetentionSec=0
```

```sh
sudo rm /etc/resolv.conf
sudo service systemd-resolved restart
sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo cat /etc/resolv.conf
```

verify

```sh
dig +trace apple.com
dig +trace baidu.com
dig +trace acme-v02.api.letsencrypt.org
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
