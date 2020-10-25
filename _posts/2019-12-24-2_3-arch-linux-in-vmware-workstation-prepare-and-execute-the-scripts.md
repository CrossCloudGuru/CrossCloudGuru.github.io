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

1. Accessing the VM from a terminal
1. Synchronize the time
1. Download the repository
1. Install git
1. Clone the git repository
1. Prep and execute the scripts
1. Admire the results

### Accessing the VM from a terminal

Now we can ssh into this virtual machine from our host machine:

```bash
ssh root@172.16.157.183
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm01.png)

### Synchronize the time


If you want to time how long the installation script takes in your configuration, make sure your time is synchronized before running the script:

```bash
timedatectl set-ntp true
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm02.png)

### Download the repository

Download the latest repository files:

```bash
pacman -Sy
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm03.png)

**Note:** I have omitted the option u as I do not want to update the live system.

### Install git

Install git:

```bash
pacman -S git
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm04.png)

Press Y to continue the installation…

### Clone the git repository

Clone the git repository:

```bash
git clone https://github.com/CrossCloudGuru/ArchInstall.git
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm05.png)

### Prep and execute the scripts

Change into the cloned repository and make the scripts executable:

```bash
cd ArchInstall
chmod +x *.sh
ll
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm06.png)

Now the fun part starts. We are ready to start the script! Lets time it:

```bash
time ./ArchInstallation.sh
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm07.png)

### Admire the results

Once completed, you can see the time it took:

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm08.png)

It shows 1 minute and 6 seconds…

Different factors influence the duration of course: internet speed, power of your host system, the mirror used and its speed…

To finalize reboot the machine and have a look at the result:

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm09.png)

It shows the GNU GRUB boot loader. That is a pleasure to my eye to see this worked …

Once booted and logged in get the details of the kernel:

```bash
uname -a
```

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm10.png)

It shows that the latest kernel (as of writing this article) is used.

**NOTICE:** I will not go in any detail of what to do next with the VM to make it a production ready and secured system. That is outside the scope of the series of articles. The sshd service is enabled and started as specified in the script. Before you can connect to it, you have to either allow root to login with a password or create an administrative account with ssh rights.
 {: .notice--warning}
 
 ![alt]({{ site.url }}{{ site.baseurl }}/assets/images/articles/ArchTerm11.png)

### Conclusion / Next Steps

This concludes this article. If all went fine, you have now a virtual machine running a freshly installed Arch Linux system and ready to connect from the host machine. To continue to the next article in this series click here: Article 3: The scripts in detail

Navigation:

* Overview
* Article 1: Setup the virtual machine 
* Article 2: Prepare and execute the scripts
* Article 3: [The scripts in detail]({{ site.baseurl }}{% post_url 3_3-arch-linux-in-vmware-workstation-the-scripts-in-detail %})

[The scripts in detail](https://blog.crosscloud.guru/blog/tutorial/3_3-arch-linux-in-vmware-workstation-the-scripts-in-detail)


bla bla
