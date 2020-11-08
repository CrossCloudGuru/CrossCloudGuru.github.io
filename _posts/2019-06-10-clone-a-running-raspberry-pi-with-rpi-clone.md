---
title: "Clone a running Raspberry Pi with rpi-clone"
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
excerpt: "With rpi-clone it is really simple to clone a running Raspberry Pi onto a second SD Card."
---

Recently I encountered the fact that I could not clone the SD card of my working Raspberry Pi due to a size mismatch of a few MBs. Forcing the cloning rendered the target SD card unable to boot. So I needed to come up with an alternative solution. I have found rpi-clone and this article will cover how I used it.

You can find [rpi-clone here]. It has an extensive README for different options. I will describe here the steps taken to clone a running Raspberry Pi to new SD card.

### Prerequisites

* A running Raspberry Pi ready configured
* A SD card adapter to plug in the Raspberry Pi
* A target SD card with enough capacity to hold the running configuration
* Internet connectivity from the Raspberry Pi

Now we go over the steps.

### Step 1. Get rpi-clone on the Raspberry Pi

First we need to get the scripts used on the Raspberry Pi and get the files where they can be executed.

```bash
# Clone the repository with git:
git clone https://github.com/billw2/rpi-clone.git

# Move into the folder:
cd rpi-clone

# Make the commands system wide accessible:
sudo cp rpi-clone rpi-clone-setup /usr/local/sbin

# Make the commands executable:
sudo chmod +x /usr/local/sbin/rpi-clone*
```

I have inserted my second SD card into my USB SD card reader stick and plugged into the Raspberry Pi.
This second SD card in the card reader presents itself on the system as device `sda`. Make sure you figure out what the device name for your target SD card is. 

The next command I used to clone the running system to my target SD card:

```bash
sudo rpi-clone -v -f sda
```

The script asks for a confirmation and then continues. The options I choose do the following for me:

* `-v` for verbose output
* `-f` force to clone all partitions

After the cloning it asks for final confirmation to unmount the sda SD card. After that you can remove the SD card and put it in the next Raspberry Pi and it will boot.


[rpi-clone here]: https://github.com/billw2/rpi-clone
