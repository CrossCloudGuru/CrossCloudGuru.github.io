---
title: "Show the IP address at boot on the SenseHAT attached to your Raspberry Pi"
original_date_label: 2019-06-10T11:33:57-04:00
last_modified_at: 2020-11-08T21:18:50-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
  - rpi
excerpt: "In this article you will learn how to show the Raspberry Pi it's IP address at boot time until you are logged in."
---

For my demos in the field I use a Raspberry Pi with a SenseHAT attached to it and power it through my power-bank. For internet connectivity I use the Personal Hotspot feature of my iPhone.

This article will detail the steps necessary to show the IP address the Raspberry Pi uses once booted so you can easily ssh into it.

For my demos in the field I use a Raspberry Pi with a SenseHAT attached to it and power it through my power-bank. For internet connectivity I use the Personal Hotspot feature of my iPhone.

This article will detail the steps necessary to show the IP address the Raspberry Pi uses once booted so you can easily ssh into it.

### How it works

The script will install a service that starts at boot time once the network becomes active. The service calls a script that finds the IP address and calls a python script to show the IP address on the SenseHAT.

It loops showing the IP address on the SenseHAT until you login to the Raspberry Pi with SSH. The login script (.bashrc) calls a script to stop the service and clears the SenseHAT\’s display.

### Prerequisites

    You have a running Raspberry Pi with a SenseHAT attached to it.

###Optional

* The Raspberry Pi is configured for headless operation (See article: [Configuring your Raspberry Pi for headless operation])
* The Raspberry Pi is connected to a WiFi or Hotspot (See article: [Connecting your Pi to the Personal Hotspot on an iPhone])

### Steps

The files used for this are available on my GitHub repository. To download and install the service, execute the following on your Raspberry Pi:

```bash
git clone https://github.com/CrossCloudGuru/showIP_pub.git
cd showIP_pub
chmod +x install_showIP.sh
sudo install_showIP.sh
```

The script displays the progress while execution and writes it to a log file. If you want to read through the log:

```bash
cat /tmp/install_showIP.log
```

[Configuring your Raspberry Pi for headless operation]: {% post_url 2019-06-07-configuring-your-raspberry-pi-for-headless-operation %}
[Connecting your Pi to the Personal Hotspot on an iPhone]: {% post_url 2019-03-16-connecting-your-pi-to-the-personal-hotspot-on-an-iphone %}
