---
title: "[2/3] Arch Linux in VMware Workstation – Prepare and execute the scripts"
date: 2019-12-24T15:10:00-04:00
toc: true
categories:
  - blog
  - tutorial
tags:
  - linux
  - vmware
---

This article is part of the series Arch Linux in a VMware Workstation. In the second article I will explain to you how I get my scripts into virtual machine and execute them.
Configuration I use:

* Ubuntu 18.04 Desktop
* VMware Workstation Pro 15

For this article to follow along, I assume that you have completed the article Setup the virtual machine to continue.

Steps in this article: 

* Accessing the VM from a terminal.
* Synchronize the time.
* Install git
* Download the repository
* Prep and execute the scripts
* Admire the results

Now we can ssh into this virtual machine from our host machine:

```bash
ssh root@172.16.157.183
```



![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm01.png)

