---
title: "Write and backup SD cards for your Raspberry Pi"
original_date_label: 2019-06-10T15:00:00-04:00
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
excerpt: "For my demos I use a Raspberry Pi with a SenseHAT attached to it. Every time a new update of the software becomes available, I want to make a backup of my Raspberry Pi SD card before I install the latest and greatest."
---

For my demos I use a Raspberry Pi with a SenseHAT attached to it. Every time a new update of the software becomes available, I want to make a backup of my Raspberry Pi SD card before I install the latest and greatest.

With [Etcher from Balena] it is only possible to write images to a SD card, not making an Image of the SD card. Yes you can do it with DiskUtility from Apple, but recently I found an interesting tool that can do both writing and make backups of SD cards. On top of that, it has the option to compress the images what saves a ton of space on you harddrive. It is called [ApplePi-Baker by Hans Luijten] . Usage is simple:

1.  Put your SD card in your card-reader and attach it to your Mac
2.  Fire up the app, enter your password to allow it to make changes
3.  Choose what you want to do: Backup or Restore

To download the latest version: [click here].


[Etcher from Balena]: https://www.balena.io/etcher/
[ApplePi-Baker by Hans Luijten]: https://www.tweaking4all.com/ApplePi-Baker
[click here]: https://www.tweaking4all.com/ApplePi-Baker2.dmg
