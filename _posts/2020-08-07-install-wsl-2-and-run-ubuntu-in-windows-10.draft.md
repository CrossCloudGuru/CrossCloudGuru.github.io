---
title: "Install WSL 2 and run Ubuntu in Windows 10"
original_date_label: 2020-08-07T15:00:00-04:00
last_modified_at: 2020-11-01T16:17:45-04:00
#classes: wide
#toc: false                # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
excerpt: "This is the summary of the article as shown on the pages of the blog."
---

In this article I take you through the steps I took to install WSL 2 in a Windows 10 virtual machine and download a Ubuntu 20.04 Linux image to get started. I use VMware Fusion on a Mac.

Microsoft has overhauled their Windows Subsystem for Linux (aka WSL) recently. There are some significant updates in this update. One of them is that it runs with an acutal Linux kernel. Build by Microsoft, but anyway. It is time to test is out. WSL can be installed on most Windows 10 flavours but not all. Check upfront if your version is suitable.

As test machine I have cloned an up-to-date Windows 10 Enterprise VM. While still powered off I have selected the "Enable hypervisor applications in this virtual machine"

Open the settings for your virtual machine and select "Processors & Memory"

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm02.png)


[FusionHypervisorSettings.jpg](https://github.com/CrossCloudGuru/CrossCloudGuru.github.io/blob/master/assets/images/articles/2020-08-07-wslOnWin10/FusionHypervisorSettings.jpg)