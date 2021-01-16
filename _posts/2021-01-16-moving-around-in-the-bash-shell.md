---
title: "Moving around in the bash shell"
original_date_label: 2021-01-16T16:00:00-04:00
last_modified_at: 2021-01-16T16:00:00-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
excerpt: "Figuring out how to navigate in the bash shell? This article will explane some need to know moves."
---

Spending a lot time in the bash terminal asks for navigation optimizations while editing commands. This article will take you through my favourites. At the end I have some interesting links to more information to the topic.

The below picture is a perfect visual reminder for me that I have found on the internet:

![Basic bash navigation movements](https://raw.githubusercontent.com/CrossCloudGuru/CrossCloudGuru.github.io/master/assets/images/articles/2021-01-16-moving-around-bash/moving_bash-1024x287.png)

### My Favourites

| Key Combo | Description |
| :--- | :--- |
| Ctrl + u & sudo & Ctrl + y | Cut all before cursor. Type sudo. Paste back cut text (yank) |
| sudo !! | Thpe sudo !! where !! repeats the last run command |

### Move cursor

| Key Combo | Description |
| :--- | :--- |
| Ctrl + a | Go to the beginning of the line (Home) |
| Ctrl + e | Go to the End of the line (End) |
| Alt + b | Back (left) one word |
| Alt + f | Forward (right) one word |
| Ctrl + f | Forward one character |
| Ctrl + b | Backward one character |
| Ctrl + xx | Toggle between the start of line and current cursor position |

### Move cursor

| Key Combo | Description |
| :--- | :--- |
| Ctrl + u | Cut the line before the cursor position |
| Alt + Del | Delete the Word before the cursor |
| Alt + d | Delete the Word after the cursor |
| Ctrl + d | Delete character under the cursor |
| Ctrl + h | Delete character before the cursor (backspace) |
| Ctrl + w | Cut the Word before the cursor to the clipboard |
| Ctrl + k | Cut the Line after the cursor to the clipboard |
| Alt + t | Swap current word with previous |
| Ctrl + t | Swap the last two characters before the cursor (typo) |
| Esc + t | Swap the last two words before the cursor. |
| Ctrl + y | Paste the last thing to be cut (yank) |
| Alt + u | UPPER capitalize every character from the cursor to the end of the current word. |
| Alt + l | Lower the case of every character from the cursor to the end of the current word. |
| Alt + c | Capitalize the character under the cursor and move to the end of the word. |
| Alt + r | Cancel the changes and put back the line as it was in the history (revert) |
| Сtrl + _ | Undo |

### History

| Key Combo | Description |
| :--- | :--- |
| Ctrl + r | Recall the last command including the specified character(s)(equivalent to : vim ~/.bash_history). |
| Ctrl + p | Previous command in history (i.e. walk back through the command history) |
| Ctrl + n | Next command in history (i.e. walk forward through the command history) |
| Ctrl + s | Go back to the next most recent command. |
| Ctrl + o | Execute the command found via Ctrl+r or Ctrl+s |
| Ctrl + g | Escape from history searching mode |
| Alt + . | Use the last word of the previous command |

You notice that some of the commands require the ALT key. There is no ALT key on a Mac keyboard. But there is an Option key. To use the Option key with the functionality of the ALT key there need to be some changes to the settings of the Terminal you use. Let me show below for the default Mac Terminal and also the iTerm2 Terminal. The ALT key is also referred to as Meta key or as ESC+ key.

For the default Terminal open the preferences and follow the steps:

* Select the profile you are using.
* Select Keyboard on the right
* Select Use Option as Meta key

![MacOS terminal settings for Alt key](https://raw.githubusercontent.com/CrossCloudGuru/CrossCloudGuru.github.io/master/assets/images/articles/2021-01-16-moving-around-bash/SettingsDefTerminal-1-1024x902.jpg)

For iTerm2 the steps are similar. Open the preferences and follow the steps:

* Select the profiles menu.
* Select the default profile you use.
* Click on the Keys configuration on the right.
* Select for both Option keys the Esc+ option on the right. (Here selecting Meta does not get the result we are looking for.

![MacOS iTerm2 settings for Alt key](https://raw.githubusercontent.com/CrossCloudGuru/CrossCloudGuru.github.io/master/assets/images/articles/2021-01-16-moving-around-bash/SettingsiTerm2-1-1024x587.jpg)

### Sources

* [Bash Shortcuts For Maximum Productivity – Skorks](https://skorks.com/2009/09/bash-shortcuts-for-maximum-productivity/)
* [fliptheweb/bash-shortcuts-cheat-sheet: Useful shortcuts for bash/zsh](https://github.com/fliptheweb/bash-shortcuts-cheat-sheet)
