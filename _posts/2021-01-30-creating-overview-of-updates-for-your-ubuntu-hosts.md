---
title: "Creating overview of updates for your Ubuntu hosts"
original_date_label: 2019-07-06T15:00:00-04:00
last_modified_at: 2020-10-31T23:28:02-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
  - debian
  - ubuntu
excerpt: "Having multipe debian based systems you want to update and want to check what updates are available? This article will tell you how to get lists."
---

Recently I had a situation where I wanted to have an overview of all security updates there are available for my Ubuntu servers. For that I have created the script I will explain here in this article.

In the end you will need to have two files:
1. The script
2. Input file

The script is where all the magic happens and the input file contains the array of hosts that need to be checked. That was a requirement to my original challence as there are multiple environments to check and with that the reporting will be grouped per environment.

When the script is run, it will generate per host an output file with all the security upates for that host, it will genarate a file that lists all hosts that have a reboot already pending and it will generate a file combining all the information from the previous outputs.

Assumptions:

* For authentication I assume SSH keys are used to access the targeted hosts.
* The ssh key should already be loaded to your ssh-agent before executing the script.
{: .notice}

clear clears your screen if this is possible, including its scrollback buffer (if the extended “E3” capability is defined).
{: .notice--info}

Tip:
If you have multiple keys you require for accessing different environments/hosts, read this article that I have here.
Menu for selecting SSH keys to load - CrossCloud.Guru Blog
https://blog.crosscloud.guru/blog/tutorial/menu-for-selecting-ssh-keys-to-load/






Find it on GitHub:  
[CrossCloudGuru/menu-for-selecting-ssh-keys-to-load: Menu for selecting SSH keys to load](https://github.com/CrossCloudGuru/menu-for-selecting-ssh-keys-to-load)

