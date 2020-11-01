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
excerpt: "Running Windows 10 and want to have the goodness of Ubuntu Linux at your fingertips, this article describes how to install WSL and Ubuntu on Windows 10."
---

In this article I take you through the steps I took to install WSL 2 in a Windows 10 virtual machine and download a Ubuntu 20.04 Linux image to get started. I use VMware Fusion on a Mac.

Microsoft has overhauled their Windows Subsystem for Linux (aka WSL) recently. There are some significant updates in this update. One of them is that it runs with an acutal Linux kernel. Build by Microsoft, but anyway. It is time to test is out. WSL can be installed on most Windows 10 flavours but not all. Check upfront if your version is suitable.

### Step 1. Prepare virtual machine

As test machine I have cloned an up-to-date Windows 10 Enterprise VM. While still powered off I have selected the "Enable hypervisor applications in this virtual machine"

Open the settings for your virtual machine and select "Processors & Memory"

![Virtual machine settings - proccessors & memory]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/FusionHypervisorSettings01.jpg)

Next click on "Advanced options" to see additional options we are looing for Here you can click the tick-box for "Enable hypervisor applications in this virtual machine"

![Advanced settings - Enable hypervisor applications]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/FusionHypervisorSettings02.jpg)

### Step 2. Install WSL on Windows 10

Now the virtual machine can be started. Once started, log in.

To install WSL I have used PowerShell to complete the tasks.

Open PowerShell as `Administrator` and run the following commands.

- Enable WSL (Windows Subsystem for Linux)

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

- Set WSL version 2 default

```
wsl --set-default-version 2
```

- Informative message

```
F o r   i n f o r m a t i o n   o n   k e y   d i f f e r e n c e s   w i t h   W S L   2   p l e a 
s e   v i s i t   h t t p s : / / a k a . m s / w s l 2
```

Now I rebooted the virtual machine.

### Step 3. Install Ubuntu on Windows 10

After logging back in, open Microsoft Store, search for Ubuntu, and select the "Ubuntu 20.04 LTS" app:

![Microsoft Store - Searching for Ubuntu]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/msStoreUbuntu01.jpg)

### Step 4. Launching Ubuntu

Now we want to start Ubuntu. There are two options. Right after the installation from the Microsoft Store or further down the road from the Start Menu.

- Launch Ubuntu from the Microsoft Store

![Launching Ubuntu - From the Microsoft Store]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/win10WSL2-Ubuntu00.jpg)

- Launch Ubuntu from the Start menu

![Launching Ubuntu - From the Microsoft Store]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/win10WSL2-Ubuntu01.jpg)


### The Result

It might take a while the first run as some stuff has to be installed and configured in the background. But if everything went fine, you should see the terminal prompt and check what is running.

![Launching Ubuntu - From the Microsoft Store]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2020-08-07-wslOnWin10/win10WSL2-Ubuntu02.jpg)

