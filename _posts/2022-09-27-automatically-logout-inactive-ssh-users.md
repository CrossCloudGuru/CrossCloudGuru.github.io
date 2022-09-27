---
title: "Automatically logout inactive SSH users"
original_date_label: 2022-09-27T14:27:54-04:00
last_modified_at: 2022-09-27T14:27:54-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - ssh
excerpt: "How to configure SSH to automatically logout stale SSH sessions."
---

# Automatically logout inactive SSH users

Working in IT has taught me to lock my screen on my PC, Laptop or any other device. This habit has become a second nature that even at home I lock my systems.
A few systems I consider to have production style services, I configured them to automatically log out when I forget to do so. I will describe here in this article how I configured this.

In this approach, we will only making the SSH session users to log out after a particular period of inactivity.

Edit **`/etc/ssh/sshd_config`** file:

```bash
$ sudo vi /etc/ssh/sshd_config
```

Add/modify the following lines:

```bash
ClientAliveInterval 1800
ClientAliveCountMax 0
```

Press **`ESC`** key and type **`:wq`** to save and close this file. Restart sshd service to take effect the changes.

```bash
$ sudo systemctl restart sshd
```

Now, ssh to this system from a remote system. After 100 seconds, the ssh session will be automatically closed and you will see the following message:

```bash
$ Connection to 192.168.122.181 closed by remote host.
Connection to 192.168.122.181 closed.
```

From now on, whoever access this system from a remote system via SSH will automatically be logged out after an inactivity of 1800 seconds, or 30 minutes.

---
Source and Credits:
[Auto Logout Inactive Users After A Period Of Time In Linux - OSTechNix](https://ostechnix.com/auto-logout-inactive-users-period-time-linux/)
