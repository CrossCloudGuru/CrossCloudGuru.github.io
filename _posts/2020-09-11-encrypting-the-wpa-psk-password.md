---
title: "Encrypting the WPA PSK password"
date: 2020-09-11T15:00:00-04:00
excerpt_separator: "<!--more-->"
toc: true
toc_icon: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - rpi
  - netboot
---

There are a lot of things that can be done to make more difficult for people with malicious intent.

One of the golden rules concerning security is not to store passwords in plain text format.

This article will explain how to remove the plain text passwords from the wpa_supplicant.conf file used for making connections to wireless networks.

```bash
wpa_passphrase "<The SSID>" "<The PSK (password)>" | sudo tee -a /etc/wpa_supplicant/wpa_supplicant.conf
```

What happens here is:

* `wpa_passphrase` combined with the inputs for the SSID name and its PSK password to convert a WPA passphrase and SSID to the 256-bit pre-shared (“raw”) key used for key derivation.
* `sudo tee -a` sends the output piped through `|` with elevated sudo privileges using `tee -a` to append to the `wpa_supplicant.conf` file


When you pay attention you notice that the plain text password is also part of the output. For testing purposes to see if it works is fine. But once confirmed that you can connect with the converted password, you should remove this line starting with `#psk="some password"`.

```bash
sudo sed -i '/#psk=/d' /etc/wpa_supplicant/wpa_supplicant.conf
```

If you want to compare outputs from this command line tool and one available online, check this out:

Wireshark · WPA PSK Generator [https://www.wireshark.org/tools/wpa-psk.html](https://www.wireshark.org/tools/wpa-psk.html)
