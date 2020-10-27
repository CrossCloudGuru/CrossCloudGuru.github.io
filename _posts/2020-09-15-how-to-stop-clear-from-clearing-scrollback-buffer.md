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
---

This is a scripted version from my previous article Avoid clear to clear scrollback buffer of the terminal. The breakdown of this script will be discussed in this post.

Below I will go step by step through the script. At the bottom of the article you can find the whole script.

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


