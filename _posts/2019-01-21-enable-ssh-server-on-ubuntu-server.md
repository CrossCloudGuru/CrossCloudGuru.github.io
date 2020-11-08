---
title: "Enable SSH Server on Ubuntu Server"
original_date_label: 2019-07-06T15:00:00-04:00
last_modified_at: 2020-10-31T23:28:02-04:00
#classes: wide
toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
#excerpt: "This is the summary of the article as shown on the pages of the blog."
---

When doing a minimal installation of a Ubuntu server as expected you need to install some additional packages to have a functional system. One of the most important ones is the SSH server. To install the SSH server:

```bash
sudo apt-get install openssh-server
```

After installation, check the status if the service is running:

```bash
sudo systemctl status ssh
```

![Ubuntu Terminal - systemctl status ssh]({{ site.url }}{{ site.baseurl }}/assets/images/articles/2019-01-21-SSHonUbuntu/statusSSHonUbuntu01.jpg)

```bash
sudo nano /etc/ssh/sshd_config
```

When finished, make sure to restart the service to apply the changes made to the config file:

```bash
sudo systemctl restart ssh
```
