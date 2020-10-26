---
title: "[0/3] Arch Linux in VMware Workstation – Overview"
date: 2019-12-24T15:05:00-04:00
toc: true
toc_icon: list-alt

categories:
  - blog
  - tutorial
tags:
  - linux
  - vmware
---

### Introduction

This article is the home of the series Arch Linux in VMware Workstation.

In these series I will explain to you how I managed to install Arch Linux as a guest in VMware Workstation (on Linux) mostly automated with an installation script.

I got curious about Arch Linux after reading an interesting article in one of my favorite magazines. The trigger for me was that Arch Linux uses a rolling update philosophy. So let’s dig in and get it installed in a virtual machine and play with it to figure out the differences to my Ubuntu addiction. As usual it did not fully work for me right away. You know, just give it a shot with only taking a glimpse of the documentation. So some more searching the web and try the instructions I found that tried to achieve similar to my goal. Still no luck. My issues were with installing the boot loader at the end of the installation. But after many reads and alternative boot loaders I got grub working.

Meanwhile I captured all the commands in the order they needed to be executed. After so many installations I created a script so the repetition would be automated up to the point of where I found challenges and once resolved I had a fully working script.

Now I can install a fresh Arch Linux installation in a VMware virtual machine ready to reboot into a functional virtual machine in about 1 minute and 5 seconds over and over again. Isn’t that cool? If you are up to it to give it a go, I invite you to the articles in the series:


### Articles  

- [Overview]({% post_url 2019-12-24-0_3-arch-linux-in-vmware-workstation-overview %})
- [Article 1: Setup the virtual machine]({% post_url 2019-12-24-1_3-arch-linux-in-vmware-workstation-setup-the-virtual-machine %})
- [Article 2: Prepare and execute the scripts]({% post_url 2019-12-24-2_3-arch-linux-in-vmware-workstation-prepare-and-execute-the-scripts %})
- [Article 3: The scripts in detail]({% post_url 2019-12-24-3_3-arch-linux-in-vmware-workstation-the-scripts-in-detail %})

Here is the link to the full scripts:
[Git repository ArchInstall scripts](https://github.com/CrossCloudGuru/ArchInstall)

---

### Sources

Next steps in the journey with Arch: General recommendations – ArchWiki  
[https://wiki.archlinux.org/index.php/General_recommendations](https://wiki.archlinux.org/index.php/General_recommendations)

To figure out the fastest mirror close to you: Arch Linux – Mirror Status  
[https://www.archlinux.org/mirrors/status/](https://www.archlinux.org/mirrors/status/)

Or generate a mirror list by country (not used in the script): Arch Linux – Pacman Mirrorlist Generator:  
[https://www.archlinux.org/mirrorlist/?country](https://www.archlinux.org/mirrorlist/?country)

If you want to read more on chroot: chroot – Wikipedia  
[https://en.wikipedia.org/wiki/Chroot](https://en.wikipedia.org/wiki/Chroot)

This is the article where I learned how to start a script in the chroot environment: [solved] a simple installation script not working / Installation / Arch Linux Forums  
[https://bbs.archlinux.org/viewtopic.php?id=204252](https://bbs.archlinux.org/viewtopic.php?id=204252)
