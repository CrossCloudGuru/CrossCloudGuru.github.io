---
title: "Configuring your Raspberry Pi for headless operation"
original_date_label: 2019-06-10T15:00:00-04:00
last_modified_at: 2020-11-08T21:50:02-04:00
#classes: wide
toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
  - rpi
excerpt: "Working with a Raspberry Pi usually does not require a keayboard, mouse and display for me. This article will learn you how to modify an OS image that will be capably booting into a working system."
---

It is not always convenient to have an additional display keyboard and maybe a mouse occupying desk space. If you are like me, most I do is via a terminal session anyway. If needed I can start the X (graphical) environment and use VNC to have my desktop displayed in a window on my Mac (or Windows for that matter).

There are basically 2 important modifications you need to make where the first is a necessity and the second a must/nice to have.

1. The first modification will enable the SSH server to be activated right after installation so it is accessible over the network.
2. The second modification enables the connectivity to your WiFi when not using a cabled network.

For this example I have downloaded the latest version of the [Raspbian image] from the [Raspberry Pi OS website]. Once downloaded, double click the downloaded zip to extract. Then double click to mount the image.

MacOS will not recognize the Linux file system. For this article that is ok. We need the partition called boot anyway.

To have the system recognize that it needs to enable the ssh-server, we need to create an empty file called ssh on the boot partition. Open a terminal and do the following:

```bash
touch /Volumes/boot/ssh
```

That’s it. Now the empty file ssh is created and the Raspberry Pi will know to enable the ssh-server to be ready waiting for your connection.

For the Raspberry Pi to know what WiFi network to connect to, we need to create a file called `wpa_supplicant.conf` and put information in it. In the terminal do the following:

```bash
touch /Volumes/boot/wpa_supplicant.conf
nano /Volumes/boot/wpa_supplicant.conf
```

Add the following to the wpa_supplicant.conf file with modifications to match your WiFi configuration:

```bash
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid=NETWORK-NAME
    psk=NETWORK-PASSWORD
}
```

How to connect to the Personal Hotspot on your iPone, [see this article].

Now with the two modifications it is time to unmount the image and write it to the SD card. How to do this, see [my article writing and making backups of SD cards].

Once written the image to the SD card, put it in the Raspberry Pi and boot it. SSH will be automatically enabled and the Pi will connect to your WiFi.



[Raspbian image]: https://downloads.raspberrypi.org/raspbian_latest
[Raspberry Pi OS website]: https://www.raspberrypi.org/downloads/raspberry-pi-os/
[see this article]: {% post_url 2019-03-16-connecting-your-pi-to-the-personal-hotspot-on-an-iphone %}
[my article writing and making backups of SD cards]: {% post_url 2019-06-07-write-and-backup-sd-cards-for-your-raspberry-pi %}

