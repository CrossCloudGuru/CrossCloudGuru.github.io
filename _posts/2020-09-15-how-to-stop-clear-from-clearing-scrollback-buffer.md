---
title: "How to stop clear from clearing scrollback buffer"
date: 2020-09-15T15:15:00-04:00
#classes: wide
toc: true
toc_icon: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
excerpt: "This article will show how to disable the behaviour of clear avoiding to erase this history of your terminal output."
---

When working in a terminal shell trying and testing it can be very usefull to scroll back through all output that has passed by. Still you want to have an empty terminal from time to time when starting your next test. Usually you do this with `Ctrl-L` or with `clear`. There is an important difference between the two.

Although the option "limit scrollback to" is unchecked the terminal preferences, it is not consistent in keeping this promise when using the previous commands. This is because the command `clear` has it’s own opinion about this what to do. 

This article will describe how to make adjustments to the terminal settings in the shell to have `clear` behave like `Ctrl-L`. In this case: leave the scrollback history alone and show all when scrolling back. 

For repeatable success, I have created a script for it that I will explain in detail here. At the bottom of the article you can find the whole script.

## The script step by step

### Step 1. Store the current date and time for consumption

```bash
# Current date and time of script execution
DATETIME=`date +%Y%m%d_%H%M&`
```

### Step 2. Display terminal type

```bash
# Display current terminal type:
echo $TERM
```

Always nice to see what we have got.
Output:

```bash
$ echo $TERM
xterm-256color
```
### Step 3. Backup the current terminal settings

```bash
# Store the current terminal settings in a backup file
infocmp -x $TERM > terminal-settings.backup-${DATETIME}
```

### Step 4. Create second copy to work with

```bash
# Store the current terminal settings in a file to work with
infocmp -x $TERM > terminal-settings.new-${DATETIME}
```

### Step 5. Value of setting
From this point it gets a little more tricky.

If you type `man clear` you will see that the manual states:

> clear clears your screen if this is possible, including its scrollback buffer (if the extended “E3” capability is defined).
{: .notice--info}

We are going to remove this E3 capability and we store the specific string in a variable. The command `sed` recognizes the characters `\` and `[` as special characters and need to be escaped with an additional `\` to take the characters in the string literally.

```bash
# Store the value to search for in variable
# We need to remove the string: 'E3=\E[3J, ' but it contains the \ and [ special
# characters. These need to be escaped with an extra \
VALUE='E3=\\E\[3J, '
```

### Step 6. Search for and remove the string that controls the setting

Now that we are on the topic how sed handles special characters, it is also a challenge to work with variables. In this case the variable is incorporated and with the additional escaped characters in the variable it will consume the string to search for as we want it to.

```bash
# Search for variable in file and remove it from file
sed -i '/'"$VALUE"'/s///g' terminal-settings.new-${DATETIME}
```

### Step 7. Load the settings into the configuration
The now modified file needs to be loades as the new settings.

```bash 
# Load modified termininfo and store it
sudo tic -x terminal-settings.new-${DATETIME}
```

### Step 8. Restart the terminal to take changes active
Once back with a new fresh terminal session you will keep your whole screen history even when you type `clear`.

## The full script

```bash
#!/usr/bin/env bash
################################################################################
# Script name: keepScrollbackBuffer.sh
# Version: 1
# Author: Marco Tijbout - CrossCloud.Guru
#
# History:
#   1 - Initial released version
#
# Purpose:
#   Modify the terminal settings to avoid clear to clear the scrollback buffer
#
# Usage:
#   - Run this script on linux flavoured system with the bash shell.
#   - Settings are set globally for the system. Remove sudo from 'sudo tic' to
#   make it user bound.
#
# Improvement ideas:
#   -
################################################################################

# Current date and time of script execution
DATETIME=`date +%Y%m%d_%H%M`

# Display current terminal type:
echo $TERM

# Store the current terminal settings in a backup file
infocmp -x $TERM > terminal-settings.backup-${DATETIME}

# Store the current terminal settings in a file to work with
infocmp -x $TERM > terminal-settings.new-${DATETIME}

# Store the value to search for in variable
# We need to remove the string: 'E3=\E[3J, ' but it contains the \ and [ special
# characters. These need to be escaped with an extra \
VALUE='E3=\\E\[3J, '

# Search for variable in file and remove it from file
sed -i '/'"$VALUE"'/s///g' terminal-settings.new-${DATETIME}

# Load modified termininfo and store it
sudo tic -x terminal-settings.new-${DATETIME}

# Restart the terminal to take changes into effect.
```

Find it on GitHub:
[CrossCloudGuru/how-to-stop-clear-from-clearing-scrollback-buffer: How to stop clear from clearing scrollback buffer](https://github.com/CrossCloudGuru/how-to-stop-clear-from-clearing-scrollback-buffer)
