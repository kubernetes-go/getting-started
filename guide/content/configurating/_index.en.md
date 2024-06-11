---
title: Configuring Node 
weight: 2
chapter: true
pre: "<b>2. </b>"
---

## General

### Enable `net.ipv4.ip_forward`

```sh
sudo sh -c "echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf"
sudo sysctl -p
```

### Disable swap 

```sh
sudo vi /etc/fstab
```

comment out or remove anything like `/swapfile swap swap defaults 0 0`.

### Then please go next step with the following links:

{{% children  %}}