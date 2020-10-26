---
title: "[1/3] Netboot Pi - Setup PXE and NFS on PiServer"
date: 2020-09-15T15:14:00-04:00
excerpt_separator: "<!--more-->"
toc: true
toc_icon: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - rpi
  - netboot
---

As part of my learning project on how to run a Kubernetes cluster on Raspberry Pi’s, one of the goals is to have the Raspberry Pi’s boot over the network to get their OS image.

This article assumes that you already have a Raspberry Pi that boots from an USB SSD drive, but it is not a hard requirement. Keep in mind, part of the whole project is to create a more robust environment with fault tolerance built in. In this case, a SSD can handle many more read and write operations before failures are likely to happen compared to the SD-Cards commonly used.

## Installation and configuration

### Step 1. Update the Raspberry Pi

```bash
# Update and upgrade the rPI
sudo apt update && sudo apt full-upgrade -y
```

### Step 2. Create required directories

```bash
# Create required directories
[ ! -d /tftpboot ] && mkdir -p /tftpboot
[ ! -d /nfs ] && mkdir -p /nfs
```

### Step 3. Install required software packages if not already installed

```bash
# Install required software when missing:
[ ! -x /usr/bin/unzip ] && sudo apt install -y unzip
[ ! -x /sbin/kpartx ] && sudo apt install -y kpartx
[ ! -x /usr/sbin/dnsmasq ] && sudo apt install -y dnsmasq
[ ! -x /usr/sbin/tcpdump ] && sudo apt install -y tcpdump

sudo service nfs-kernel-server status > /dev/null
STATUS=$?
if [ ${STATUS} -ne 0 ]; then
    if [ ${STATUS} -eq 3 ]; then
        echo -e "nfs-kernel-server is installed but not running"
    else
        sudo apt install -y nfs-kernel-server
    fi
fi
unset STATUS
```

### Step 4. Make sure the required services are enabled for automatic starting at boot time and are started

```bash
# Enable and restart rpcbind and nfs-kernel-server services:
sudo systemctl enable nfs-kernel-server
sudo systemctl restart rpcbind
sudo systemctl restart nfs-kernel-server
```

### Step 5. Reconfigure dnsmasq to server TFTP files only to Raspberry Pi instances

```bash
echo 'dhcp-range=10.16.0.0,proxy' | sudo tee -a /etc/dnsmasq.conf
echo 'log-dhcp' | sudo tee -a /etc/dnsmasq.conf
echo 'enable-tftp' | sudo tee -a /etc/dnsmasq.conf
echo 'tftp-root=/tftpboot' | sudo tee -a /etc/dnsmasq.conf
echo 'pxe-service=0,"Raspberry Pi Boot"' | sudo tee -a /etc/dnsmasq.conf
```

**Note:** By adding tftp-unique-root=mac to the configuration, it will also accept the MAC address of the system that tries to boot via PXE.
{: .notice--info}

### Step 6. Enable and restart the dnsmasq service

```bash
# Enable and restart the dnsmasq service:
sudo systemctl enable dnsmasq
sudo systemctl restart dnsmasq
```

### Monitoring activity

Once you start booting your Raspberry Pi’s you probably want to watch the logs on what is happening in this space. To do so, you can watch the activity in the deamon.log.

```bash
# Watch the output of the logs while booting the Raspberry Pi
sudo tail -f /var/log/daemon.log
```

---

I hope you like the articles and learned something new.

Navigation:  

<!-- - [Overview]({% post_url 2019-12-24-0_3-arch-linux-in-vmware-workstation-overview %}) -->
- [[1/3] Netboot Pi - Setup PXE and NFS on PiServer]({% post_url 2020-09-14-1_3-setup-pxe-and-nfs-on-piserver %})
- [[2/3] Netboot Pi - Prepare netboot OS environment for Raspberry Pi’s]({% post_url 2020-09-14-2_3-prepare-netboot-os-environment-for-raspberry-pis %})
- [[3/3] Netboot Pi - Enable Raspberry Pi's to boot over the network]({% post_url 2020-09-15-3_3-enable-raspberry-pis-to-boot-over-the-network %})

Here is the link to the full scripts:
[Git repository Netboot Pi scripts](https://github.com/CrossCloudGuru/NetbootPi)
