---
title: "Cloning of Raspberry Pi SD Card images – The Next Level"
original_date_label: 2019-06-10T15:00:00-04:00
last_modified_at: 2020-11-08T21:50:02-04:00
#classes: wide
toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - experience
tags:
  - bash
  - linux
  - rpi
excerpt: "A technical article describes how to do something. This article tells about the experience I acquired that resulted into a technical solution. Here you can read my story behind the experience."
---

**Sharing my experiences: When it doesn't go as expected.**

This article is not about cloning SD cards if everything works as expected how it should work. This is an article if it doesn't go as expected, plan or hope for. Maybe not all content is newbie material, but I  encourage to go through it all as I expect you to learn as much as I did. When all material is known to you and you know a better way, I hope you have a good read.

This article has links to the more detailed technical articles of the actual steps taken.

So, recently I was traveling abroad to do a workshop at a customer site. Close to the date and already overseas I received the message that the kits the customer ordered might not be delivered in time for the workshop. At a local computer store I got a couple of Raspberry Pi’s kits with some nice sensors including SenseHATs. Back at my hotel room I needed to prepare them to work headless connecting to my phone. Of course, this did not go without a struggle and costing me a huge amount of time I did not have available. This article will describe what I faced and how I worked around it.

First challenge: Writing an existing image to a new SD card.
The first thing I encountered is that my working image for my Raspberry Pi could not be written to any other 16GB SD card I had with me because it was 17,3MB to big according to the error message. Although the image had about half of free space in it, I was not able to write the image to another 16GB SD card. When forced, waiving notifications away, it renders the second partition useless. Only to find out after a long wait for the write to complete. I only had one 32GB card with me, so I sacrificed that one, but still one to prepare.

* See article: [Writing and backup SD cards for your Raspberry Pi]

As it would be a workshop teaching the essentials, I thought it would be a good idea for a fresh start to download the latest Raspbian OS image and write that to the SD card. The auto-expand feature would fix the situation as workaround for my oversized image. Then it would be just a matter of copying over the files required.

Successfully I installed the image to the disc, added the ssh file and a wpa_supplicant.conf file with my network configuration to the boot partition on the SD card and the Pi was able to connect to my iPhone’s hotspot.
* See article: [Configuring your Raspberry Pi for headless operation]
* See article: [Connecting your Pi to the Personal Hotspot on an iPhone]

Next challenge: What is the IP address assigned to my Pi?
There is no way to find on your iPhone what IP addresses it has assigned. From previous endeavours I already knew it would assign an address in the 172.20.10.x range. I tried to use from my laptop Angry IP Scanner to find any active IP addresses on the network. It found 120+. So old school trying to ssh to any address. Usually the iPhone does not assign addresses from the high end or random. So, after the 6th attempt I found it. 0 is the network, 3 was assigned to my laptop so 8 was the right one.

As I do demos with Raspberry Pi’s with a SenseHAT attached to it, I have created a script that would display the current IP address on the SenseHAT’s display. To make it effective for that I create a service that starts as soon as the Raspberry Pi is connected to a network. It stops as soon as I log in to my Pi. My login script stops the service. Now I needed to get this on the fresh vanilla installation going. I copied the required files over to the correct locations. Guess what, the service does not start. After checking the number of files, their references, the permissions and even the content of all the files I still did not get it to work. Arrgh. Now what. Downloading an older image would again take a lot of time. Wi-Fi in hotel usually works fine for office stuff but not for downloading OS image files.
* See article: [Show the IP address at boot on the SenseHAT attached to your Raspberry Pi]

Back to the first challenge: How to fit an oversized image file?
Googling around gave me a site with a nice procedure: Create an image from your SD card. Use gparted to shrink the partition in your image and then shave off the excessive size of the image. That could work and solve my problem. So, I tested first with the image from the freshly installed Pi. It worked like a charm and could successfully write the image to the SD card. Next was to try this on the image from my original (oversized) Pi. Guess what, it crashes with an error when trying to save the new partition size. Back to square one.

It is still not totally clear if the SD card has a partition issue, or some sort of corruption on it that is transferred into the image as well. Another option is that not all 16GB SD cards from different batches and/or brands are exactly the same in size regarding the total number of MBs.

Suddenly I remembered something about RPI-Clone. Searching through some of my notes, I found the name and with Google I found it. I have seen it, but never got to it to have a closer look. What it provides is a way to copy a running configuration to any (as long it can handle your currently used space) to a smaller storage medium you have available and attached to your Raspberry Pi. This could work.
* See link: [rpi-clone]

Next error I encountered is that RPI-Clone did not want to proceed with my SD card as it recognized as the running system. It would proceed on my regular USB stick. So most likely it does not like the fact that the partitions and boot loader for a Raspberry Pi were already on the SD card. Now, how do you remove partitions from an SD card under OSX? With Disk Utility you can erase the partitions, but it will not let you remove partitions. And remember it was already passing 2a.m. So, I fired up a Windows virtual machine, assigned the SD card as an USB stick to the VM, opened Disk Manager from there and deleted all the partitions. Good to go now and RPI-Clone accepted my SD card.
* See article: [Clone a running Raspberry Pi with rpi-clone]



[Writing and backup SD cards for your Raspberry Pi]: {% post_url 2019-06-07-write-and-backup-sd-cards-for-your-raspberry-pi %}
[Configuring your Raspberry Pi for headless operation]: {% post_url 2019-06-07-configuring-your-raspberry-pi-for-headless-operation %}
[Connecting your Pi to the Personal Hotspot on an iPhone]: {% post_url 2019-03-16-connecting-your-pi-to-the-personal-hotspot-on-an-iphone %}
[Show the IP address at boot on the SenseHAT attached to your Raspberry Pi]: {% post_url 2019-06-10-show-the-ip-address-at-boot-on-the-sensehat-attached-to-your-raspberry-pi %}
[rpi-clone]: https://github.com/billw2/rpi-clone
[Clone a running Raspberry Pi with rpi-clone]: {% 2019-06-10-clone-a-running-raspberry-pi-with-rpi-clone %}
