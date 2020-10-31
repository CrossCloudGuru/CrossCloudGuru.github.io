---
title: "Error when updating: dpkg error processing package install-info error exit status 127"
original_date: 2019-07-06T15:00:00-04:00
date: 2020-10-31T23:28:02-04:00
#classes: wide
toc: false     # Show Table of Contents true|false
toc_icon: stream     # Specify the character to show as ToC logo
categories:
  - blog
  - tips
tags:
  - linux
  - dpkg
excerpt: "When updating Ubuntu systems this error shows from time to time. This article shows a workaround how to continue."
---

When you try to update your system you get the error message below. In my case with Ubuntu. Maybe applicable to other distros too.

Full error message:

```bash
dpkg: error processing package install-info (--configure):
 subprocess installed post-installation script returned error exit status 127
```

On AskUbuntu.com I found a solution that I have adapted that works for me:

```bash
sudo mv /var/lib/dpkg/info/install-info.postinst /var/lib/dpkg/info/install-info.postinst.bad-`date +%Y%m%d_%H%M%S`
```

After moving the file, repeat the update process again and it will work.
