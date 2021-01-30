---
title: "Creating overview of updates for your Ubuntu hosts"
original_date_label: 2021-01-30T15:00:00-04:00
last_modified_at: 2021-01-30T15:00:00-04:00
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

## Introduction
Recently I had a situation where I wanted to have an overview of all security updates available per host for my Ubuntu servers. 

There are a vast number of reasons why overviews like these are of interest when working with methodologies like ITIL, Site Reliability Engineering (SRE) or just common sense.

The goal of this article is using this example to explain how I made the script. With some imagination you can turn it to fit an other goal but with a similar approach. Let's crack on...

In the end you will need to have two files:
1. Input file
2. The script

The script is where all the magic happens and the input file contains the array of hosts that need to be checked. I had the requirement to have one script that can be used for multiple environments e.g. groups of hosts. 

When the script is executed, it will generate three types of output:
1.  Per host an output file with all the security upates for that host
2.  It will genarate a file that lists all hosts that have a reboot already pending
3.  It will generate a file combining all the information from the previous outputs into one

Assumptions:

* For authentication I assume SSH keys are used to access the targeted hosts.
* The ssh key should already be loaded to your ssh-agent before executing the script.
<!-- {: .notice} -->

**Tip:**  
If you have multiple keys you require for accessing different environments/hosts, read this article that I have written earlier:  
[Menu for selecting SSH keys to load - CrossCloud.Guru Blog](https://blog.crosscloud.guru/blog/tutorial/menu-for-selecting-ssh-keys-to-load/)
{: .notice--info}

## The input file
The input file is a file that is sourced in the script to read additional information. I use separate input files for different environments or sets of hosts I want to target with actions. How you name the input file is not important but I recomend you to use a descriptive name. Usually I call my scripts `<logical-group-naming>.env` to keep them apart and logical to find.



Find it on GitHub:  
[CrossCloudGuru/menu-for-selecting-ssh-keys-to-load: Menu for selecting SSH keys to load](https://github.com/CrossCloudGuru/menu-for-selecting-ssh-keys-to-load)

