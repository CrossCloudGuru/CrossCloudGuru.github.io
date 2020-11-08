---
title: "Connecting your Pi to the Personal Hotspot on an iPhone"
original_date_label: 2019-03-16T15:00:00-04:00
last_modified_at: 2020-11-08T21:50:02-04:00
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
excerpt: "Making your Raspberry Pi accessible when being on the road. This article will explain how to connect to your iPhone Hotspot"
---

While sitting at your favourite coffeeshop working on your laptop, you suddenly realize how nice it would be to have your Raspberry Pi with you to fiddle around a bit to learn some new stuff…

There is a simple and of course a more complex solution to this. The first part of this article will dive in to the easier approach. The one where you actually thought of the matter through upfront.  
Not like me, looking at my Pi if it was something blinking like ET’s finger, sitting alone in my hotel room without the ease and perks of being at home.

Ok, lets go for the easier approach. What you will need is:

* A Raspberry Pi that is already up and running
* Connected to your home network or WiFi
* For mobile power, a power bank. Most of the mobile office warriors like me have one to be able to survive the day on your smartphone
* An USB cable with a power button to give you control over when the Raspberry Pi is on or off
* A smartphone with mobile hotspot functionality enabled. For this article I use an iPhone with Personal Hotspot turned on

With the power bank you are not dependent on sitting too close to a wall outlet. I do not think your Raspberry Pi will drain your charged power bank before your laptop battery goes out. It also gives a more Mobile touch to the setup.

I find the USB cables with a power switch on them very handy to be able control when I actually turn on, or off the Pi. I prefer to do a graceful shutdown of the Raspberry Pi before disconnect it from the power.

Now let’s go in to the part to have the Raspberry Pi connect to the personal hotspot of your smartphone:

To have the Raspberry Pi know about your the personal hotspot and have it actively search for it at boot you have to edit the `wpa_supplicant`.conf file:

```bash
/etc/wpa_supplicant/wpa_supplicant.conf
```

and add a second entry for the personal hotspot wifi configuration:

```bash
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=US

network={
ssid=<iPhone name>
psk=<password configured for the personal hotspot>
}
network={
ssid=<your home or other wifi>
psk=<your home or other wifi password>
key_mgmt=WPA-PSK
}
```

Make sure you have your mobile wifi entry first to have the pi connect to it rather quickly to your hotspot.

Tip: When at home, make sure you have your personal hotspot disabled…

Now with this configuration amended, the Raspberry Pi will now try to connect at boot to your personal hotspot first. When you connect your laptop to this hotspot too, both will be on the same network and able to communicate to each other. To find the Raspberry Pi used a network scanner tool called [Angry IP Scanner]. With the IP address found of the Raspberry Pi you can connect via SSH to it as usual.


[Angry IP Scanner]: https://angryip.org/download
