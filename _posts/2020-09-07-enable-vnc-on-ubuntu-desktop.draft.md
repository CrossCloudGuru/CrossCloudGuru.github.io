---
title: "Enable VNC on Ubuntu Desktop"
original_date_label: 2020-09-07T15:00:00-04:00
last_modified_at: 2020-11-08T20:42:40-04:00 
#classes: wide
#toc: false                # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - vnc
  - ubuntu
  - linux
excerpt: "Enabling VNC on Ubuntu is really easy and most articles out there on the internet show you just that. If you actually want to access your Ubuntu macine over VNC there is more to it. This article will show you how."
---

In this article I will explain how I enabled VNC connectivity to my graphical Ubuntu environment.

Trying to enable and access your Ubuntu graphical environment via VNC can be a little challenging. Ubuntu takes security serious enforcing encryption for VNC connections. In my search to enable using VNC for remote management I found many suggestions that suggested you to install additional Graphical Server software etc, etc. Many of these suggestions did not work or gave me unsatisfying results.

I want to use VNC on my own LAN and not over the internet unless via a VPN. To achieve that connectivity without encryption you only need to install a little tool for ease of use for modifying one setting. The rest is out of the box in Ubuntu available.

Steps to enable VNC screen sharing:

### Step 1. Enable screen sharing through the GUI

![Ubuntu Settings - Enabling Screen Sharing]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/UbuntuShareDesktop01.jpg)

From the settings window

1.  Click on Sharing
2.  Click on Screen Sharing
3.  Select Allow connections to control the screen
4.  Under Access Options, select Require a password and provide a password
5.  Enable under Networks on what Network the screen sharing should be enabled

### Step 2. From a terminal install the dconf-editor and start it

```bash
# Installing the dconf-editor:
sudo apt update && sudo apt-get install -y dconf-editor

# Starting the dconf-editor:
dconf-editor
```

### Step 3. In the dconf-editor window natigate to the following:

* ORG | GNOME | DESKTOP | REMOTE ACCESS
* Then find the `Require Encryption` setting and **toggle it off** as shown below

Here are the pictures how it looks:

![dconf-editor - Select Org]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/dconf-editor01.jpg)

![dconf-editor - Select Gnome]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/dconf-editor02.jpg)

![dconf-editor - Select Desktop]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/dconf-editor03.jpg)

![dconf-editor - Select Remote Access]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/dconf-editor04.jpg)

![dconf-editor - Find Require Encryption toggle off]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-09-07-vncOnUbuntu/dconf-editor05.jpg)

Done. You can now open you VNC client from another computer and connect to this screen.