---
title: "Menu for selecting SSH keys to load"
date: 2020-10-27T13:41:16-04:00
toc: true
toc_icon: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - ssh
  - pki
---

### Overview
In this post I want to explain to you how and why I created a selection menu to load ssh identities of my choosing.

Using ssh keys for authentication in terminal sessions makes life simpler when setup correctly. If you are like me, you have collected a bunch of ssh identities for work, home, test and perhaps some cloud services. Alltogether these can number up quickly. 

Having all these ssh identities added to your ssh-agent can have the disadvantage that your connection cannot establish because too many wrong keys have been presented to your targeted host before the one that opens the lock to that system is presented.

One approach is to increase the number of retries before the ssh server of your targeted system shuts the door on you. Not sustainable when using a lot of test machines that are destroyed as quickly as you have created them.

From a security perspective, I do not want ALL my identities presented to systems that I do not own/manage. Therefore I do not add ALL my identities to my ssh-agent for "just in case ...". When a "Publick key denied" message is thrown back, I will have to load the identity that enables access to that system.

Using the `~/.ssh/config` file can be of great help to assist with many hosts and identities. Although you can work with wildcards, managing your connections here can be cumbersome. 

As a solution to this dilemma I have chosen to work with a menu that presents me all the private identities I have on my system. From here I select the key I want to add and the script will start the process to add the key to the ssh-agent. If protected with a password, I will be prompted for the password to unlock the private key.

Let's dive in deeper how I added this approach to my system and how it helps me.


### The script step by step

#### Location of ssh identities

The default location of ssh identies for Linux or MacOS based systems using openssh is under your homedirectory in the hidden folder `.ssh`. In case you have a different location for your identities, you can change this value to support your needs:

```bash
SOURCE_DIR="${HOME}/.ssh"
```

#### Read files into an array

Next step is to load all private key identity files. These are filtered out

```bash
ARR_FILES=( $(grep -d skip -l "BEGIN" ${SOURCE_DIR}/*) )

```

What happens here:

* `ARR_FILES` is the variable where the output of the grep command is stored in
  * Arrays are defined by using `( )` after the `=`
* `grep` is the command used to search in the files for a specific string
  * `-d skip` specifies that directories have to be skipped
  * `-l` specifies to list the file names that contain the string we are looking for
  * `"BEGIN"` is the sting we are looking for. Any private key has in the header this word
  * `${SOURCE_DIR}/*` specifies the location where to filter the files we want to check


#### Way out of the menu

The scripts provides a menu list of all files we can work with. We need to have an option to exit the menu. So we add an additonal item to the array for that purpose.

```bash
ARR_FILES+=( "Exit selection ..." )
```

To add an additional value to the array we use the `+` before the `=` and include the additional value between `( )`.

#### Start the logic

For ease of use I use functions to make the logic easier to comprehend. I grouped the work in the function `fnCreateMenu` that is assisted by another function `fnAddSSH`.

At the bottom of the script I call the main function:

```bash
fnCreateMenu "${ARR_FILES[@]}"
```

What happens here is:

* The function `fnCreateMenu` is called
* All values of the array `ARR_FILES` are passed to it by specifying the selector `[@]` which means ALL


##### Building the menu

The function `fnCreateMenu` builds the menu. What happens I will discuss step by step:

```bash
select option; do # in "$@" is the default
```

* `select option` provides the user of the script the choice
  * `do # in "$@" is the default` where 
  * `#` is the total number of items in the array
  * `in "$@" is the default` in the list of items presented by the arry. The part "is the default" is not clear to me. 


```bash
if [ "$REPLY" -eq "$#" ];
then
    echo -e "Exiting..."
    break;
```

* This `if` statement compares if the input `"$REPLY"` is equal `-eq` to the total number of items in the array `"$#"` e.g. the last option we have added to the list of files.
* `then` if it is true
* `echo -e "Exiting..."` Echo that we will exit the loop
  * `break;` to break out of the loop

```bash
elif [ 1 -le "$REPLY" ] && [ "$REPLY" -le $(($#-1)) ];
then
    echo -e "You selected $option which is option $REPLY"
    fnAddSSH
    break;
```

* An additional condition with `elif` to select the correspondig option listed
* `then` if it is true
* Provide some output
  * Call the function `fnAddSSH` (details below)
  * When function `fnAddSSH` is completed, `break` out of the function to exit.

```bash
else
    echo -e "Incorrect Input: Select a number 1-$#"
```

* What happens `else` if the before conditions are not met
  * Provide output that an incorrect input is provided and a range to choose from `1-$#` where `#` is the total number of items in the array

```bash
    fi
done
```
 
 * `fi` to close the end of the `if` loop
   * `done` to end the `select` statement 

##### Adding the key to the ssh agent

The function `fnAddSSH` adds the selected key to the ssh agent

```bash
fnAddSSH() {
    echo -e "Adding SSH Key $option"
    ssh-add $option
}
```

* Provide what key `$option` will be added
  * where `$option` is the filename with path
* `ssh-add $option` 
  * where `ssh-add` is the actual command to add the key to the ssh agent, and
  * `$option` is the key name with full path

This concludes the explanation of the script.

### Accessible system wide

The whole purpose is making it easier to load identities to the ssh-agent. Preferably form anywhere on your system where you are currently working. For example in your GitHub repostiory somewhere in a directory tree. The best way is to add this script to a directory that is part of your `$PATH`.

When using the `bash` shell the `.bashrc` file will automatically add the directory `bin` to your `$PATH` if the folder exists under the root of your homedirectory.

```bash
cp add-ssh.sh ~/bin/add-ssh
chmod +x ~/bin/add-ssh
```

* Copy the file from the location where it is to your `bin` directory under your homedirectory `~/bin/`
* Make the file executable with `chmod +x`

Now the script can be called from anywhere on your system and will present you the options to choose from.

### The full script

```bash
#!/usr/bin/env bash

SOURCE_DIR="${HOME}/.ssh"
ARR_FILES=( $(grep -d skip -l "BEGIN" ${SOURCE_DIR}/*) )
ARR_FILES+=( "Exit selection ..." )

fnAddSSH() {
    echo -e "Adding SSH Key $option"
    ssh-add $option
}

fnCreateMenu () {
    select option; do # in "$@" is the default
        if [ "$REPLY" -eq "$#" ];
        then
            echo -e "Exiting..."
            break;
        elif [ 1 -le "$REPLY" ] && [ "$REPLY" -le $(($#-1)) ];
        then
            echo -e "You selected $option which is option $REPLY"
            fnAddSSH
            break;
        else
            echo -e "Incorrect Input: Select a number 1-$#"
        fi
    done
}

fnCreateMenu "${ARR_FILES[@]}"
```

---

```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
​```
