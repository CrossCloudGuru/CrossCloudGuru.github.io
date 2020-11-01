---
title: "Moving around in vi(m)"
original_date_label: 2020-11-01T11:33:57-04:00
last_modified_at: 2020-11-01T11:54:50-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
excerpt: "Figuring out how to navigate in vi? This article will explane some need to know moves."
---

Although there are many editors available on the Linux commandline, there are sometimes well founded reasons to edit with vi or vim. When you are not familiar with the keyboard shortcuts how to move around vi, the picture below can be of help to get started.

![Basic vi navigation movements](https://raw.githubusercontent.com/CrossCloudGuru/CrossCloudGuru.github.io/master/assets/images/articles/vim-movements-v1.png)


### Some movements

| key | action |
| :--- | :--- |
| e | move to end of word |
| 5e | move foreward to the end of the 5th word from the current position |
| w | move to next word |
| 2w | move foreward to the 2nd word from the current position |
| b | move back to beginning of word (or special character) |
| 3b | move back to second word AND character from current position |
| 0 or ^ | 0 go to beginning of line or ^ to go to first non blank character (like code identions)|
| j | move to next line |
| k | move to previous line |
| 1G or gg | 1G go to 1st line or gg |
| G | go to last line of file |


### Some editings

| key | action |
| :--- | :--- |
| o | insert line below current one and start editing|
| A | append at end of line and start editing |
| a | append at current position |
| x | delecte current character
| dw | delete current word |
| cw | change current word |
| yy | copy current line |
| p | paste current buffer |
| dd | delete current line |
| 10dd | delete 10 lines from current position |
| u | undo last action |



The above are just a few of many keybindings that vi / vim has to offer. I had this picture printed out and on my desk to get familiar. It automatically moves into the background (under a pile of rubble).