---
title: "Creating overview of updates for your Ubuntu hosts"
original_date_label: 2021-01-30T15:00:00-04:00
last_modified_at: 2021-02-19T15:00:00-04:00
#classes: wide
#toc: false               # Override from default setting: True
#toc_icon: stream         # Override from default setting: stream
categories:
  - blog
  - tutorial
tags:
  - bash
  - linux
  - debian
  - ubuntu
excerpt: "Having multipe debian based systems you want to update and want to check what updates are available? This article will tell you how to get lists."
---

## Introduction
Recently I had a situation where I wanted to have an overview of all security updates available per host for my Ubuntu servers. 

There are a vast number of reasons why overviews like these are of interest when working with methodologies like ITIL, Site Reliability Engineering (SRE) or just common sense.

The goal of this article is using this example to explain how I made the script. With some imagination you can turn it to fit another goal but with a similar approach. Let's crack on...

In the end you will need to have two files:
1. Input file
2. The script

The script is where all the magic happens and the input file contains the array of hosts that need to be checked. I had the requirement to have one script that can be used for multiple environments e.g. groups of hosts.

When the script is executed, it will generate three types of output:
1.  Per host an output file with all the security upates for that host
2.  It will genarate a file that lists all hosts that have a reboot already pending
3.  It will generate a file combining all the information from the previous outputs into one

Assumptions:
* For authentication I assume SSH keys are used to access the targeted hosts.
* The ssh key should already be loaded to your ssh-agent before executing the script.

<!-- {: .notice} -->
**Tip:**  
If you have multiple keys you require for accessing different environments/hosts, read this article that I have written earlier:  
[Menu for selecting SSH keys to load - CrossCloud.Guru Blog](https://blog.crosscloud.guru/blog/tutorial/menu-for-selecting-ssh-keys-to-load/)
{: .notice--info}

## The input file
The input file is a file that is sourced in the script to read additional information. I use separate input files for different environments or sets of hosts I want to target with actions. How you name the input file is not important but I recomend you to use a descriptive name. Usually I call my scripts `<logical-group-naming>.env` to keep them apart and logical to find.

Let me show the example input file for this article. I have named it `cloud-management-servers.env`:

```bash
# Version: 20210128-1608
# Author: Marco Tijbout
# This file contains input variables for the listSecurityUpdates.sh script.

# Hosts in the cluster specified as an array:
HOSTS=( 192.168.13.6 \
        192.168.13.8 \
        192.168.13.9 \
        192.168.13.10 \
        192.168.13.11 \
        192.168.13.14 \
        192.168.13.15 )
```

The commented lines contain information on what the purpose is of it and a version number. In this case I use a simple system to use the current date and time to show me how recent this version is.

On working with variables in bash you could write a book by it self. I will omit that.
Here I specify an array containing IP addresses of the hosts I want to check and for readability I continue on a new line after each IP address until the end of the array. The items in the array in this example are separated by a space. For readability purposes I've added line breaks '\\' to continue on a new line.

## The script

Now let us have a look at the script and break it down in chunks. I will go over each chunk separately here. Also I will explain why I include some of the code which you will find in scripts of my making.

```bash
#!/usr/bin/env bash
# Version: 20210128-1603
# Author: Marco Tijbout
#
# This script is used to generate a list of security updates that are available
# for installation on the specified hosts in the input file.
#
# Syntax:
# ./listSecurityUpdates.sh <input_file_with_list_of_servers_to_check>
```
The header with some information on the purpose and the usage of the script.

I do not like to come up with version numbers that only show a sequential number. It needs to have a little more meeting. For this purpose I use a combination of date and time in a reverse notation: `yyyymmdd-hhmm` 

Why? Because there is only one unique point in time and the chance of having multiple versions to try out on one day in one hour because you are testing it out, is more than likely to happen. This makes it easier for me to understand when I saved and commited this version and when.

```bash
# Let's begin ...
echo -e "\n\nStart processing $0"
```
This echo will show the name of the script.

Most scripts I make are also used in CI/CD pipelines where you can observe the output or logs from a new run. Hence, I want to have detailed output of progress and succes or failure. This habit makes troubleshooting so much faster as you know exactly what script at what stage has an issue.

```bash
# Current date and time of script execution
DATETIME=`date +%Y%m%d_%H%M%S`
```
Working with automation makes logging more important to learn what happened and when. For the when part I alwas use the date and time of  execution of the script. This variable is filled and stays the same during the whole script. Each script has it's own unique date and time stamp this way.

```bash
fnSucces() {
    if [ $EXITCODE -eq 0 ]; then
        echo -e "  - Succesful.\n"
    else
        echo -e "  - Failed!"
        # Consider exiting.
        echo -e "  - Exitcode: $EXITCODE"
        # exit $EXITCODE
    fi
}
```
As mentioned earlier, I like detailed information on the status. When relevant to some steps and commands of the script, I want to see the status of success or failure in the output. This function takes the value from the variable `$EXITCODE` that is passed on. Further down the script you will see how I pass this through.

```bash
# Check if an environment file is supplied on the command line.
if [ $# -eq 0 ]
then
    echo -e "No arguments supplied.\nPlease provide name of env file to process ..."
    exit 1
fi
```
This part checks if an argument is provided while executing the script. If nothing is provided, it will throw an error message and exit the script.

```bash
# Source the variables
ENV_FILE=$1
. ./"${ENV_FILE}"
echo -e "\nEnvironment file loaded: ${ENV_FILE}"
```
There the provided argument is loaded as a source to the script. The old way of doing this is with ***dot space dot slash file name*** and current days you also see often ***source file name***.

```bash
# Specify the Sysadmin Admin credential details. While executing a password 
# for the private key may be asked.
USER=remoteUserName
SSH_ID="~/.ssh/id_to_use"
ADMIN="-i ${SSH_ID} ${USER}"
echo -e "\n- Load SSH Agent ..."
eval $(ssh-agent -s)

echo -e "\n- Load SSH Key ..."
ssh-add "${SSH_ID}"
```
As we are going to access systems remotely, I mentioned already that I assume we do secure authentication with SSH keys.

This block of code specifies:
* what username to use,
* the ssh key for it,
* check if the ssh-agent is loaded
* load the ssh key to the agent

```bash
fnCombineLogs() {
    echo -e '\n# ---------------------------------------------------------------------------- #' >> security-updates-${DATETIME}-all.txt
    # REMOTE_NAME=$(dig +short -x $i) >> security-updates-${DATETIME}-all.txt
    echo -e "Hostname       : ${REMOTE_NAME}" >> security-updates-${DATETIME}-all.txt
    echo -e "IP Address     : ${i}" >> security-updates-${DATETIME}-all.txt
    echo -e "Reboot required: ${REBOOT_REQ}" >> security-updates-${DATETIME}-all.txt
    echo -e "\nSecurity updates available:" >> security-updates-${DATETIME}-all.txt
    cat ./output/security-updates-${DATETIME}-${i}.txt >> security-updates-${DATETIME}-all.txt
    echo -e "\n"
}
```
Here we find a function that takes input from the code below and adds it to a combined output file. This will generate one overview file with all the information about the array of hosts you have checked. This makes consumption easier for those who have to evaluate the contents instead of going over numerous files.

Also I would like to bring to attention that bash scripts are read sequential into memory. This means simplified that a function you want to call in a script has to be already read in to memory. Else the script will throw an error. Even if the script is fully in memory, the order of reading code is of importance too.

```bash
# For each host in the array the below actions are performed.
for i in "${HOSTS[@]}"
do
```
From the environment file passed on through the command line argument if everything is correct, the variable (array) HOSTS is read and contains now an array of ip addresses (or resolvable names) to check. These two lines will go over each item in order of the array and ***do*** something.

```bash
    echo -e "\nNow connecting to: ${i}"
```
Generate as output what host we are connecting to.

```bash
    echo -e "- Quietly updating the apt repositories ..."
    ssh ${ADMIN}@$i sudo apt-get -qq update
    EXITCODE=$?; fnSucces $EXITCODE
```
On the remote host, make sure the latests package information is loaded.

Here we also find the code:
```bash
EXITCODE=$?; fnSucces $EXITCODE
```
This puts the exit code from `$?` into a variable named `EXITCODE`. By doing so, the exitcode does not get lost as it is for each command unique.

Next the function `fnSuccess` is called and the variable `$EXITCODE` is passed along for processing.

Beacause we have put the check in to a function it saves a lot of code with the result it makes the script more readable and output uniform.

```bash
    echo -e "- Generate list of available security updates ..."
    ssh ${ADMIN}@$i "sudo apt list --upgradable | grep security > /home/${USER}/security-updates-${DATETIME}-${i}.txt"
    # ssh ${ADMIN}@$i "sudo apt list --upgradable > /home/${USER}/updates-${DATETIME}-${i}.txt"
    EXITCODE=$?; fnSucces $EXITCODE
```
After loading the fresh list of packages, list the possible updates available and fildter the updates of the type `security` and echo these to an output file.

Alternatively I've added a line to collect all updates available to use if so desired.

```bash
    # echo -e "\n- See what is in the home folder:"
    # ssh ${ADMIN}@$i ls -l /home/${USER}/
```
Optional: Have a peek in the home folder of the user on the remote host. Can come in handy if troubleshooting is required.

```bash
    echo -e "- Copy over the file to local ..."
    scp -C -i ${SSH_ID} ${USER}@${i}:/home/${USER}/security-updates-${DATETIME}-${i}.txt ./output
    EXITCODE=$?; fnSucces $EXITCODE
```
Next the output file that has been generated is copied locally in the folder output. Make sure you have a folder named output.

```bash
    echo -e "- Remove the file from host ..."
    #ssh ${ADMIN}@$i rm /home/${USER}/security-updates-${DATETIME}-${i}.txt
    ssh ${ADMIN}@$i rm /home/${USER}/security-updates-*.txt
    EXITCODE=$?; fnSucces $EXITCODE
```
After the file is copied over locally, it is removed from the remote host.

```bash
    echo -e "- Check if reboot is required ..."
    if ssh ${ADMIN}@$i stat /var/run/reboot-required \> /dev/null 2\>\&1
    then
        echo -e "  - Reboot is pending!"
        echo -e "$i" >> reboot-required-${DATETIME}.txt
        REBOOT_REQ=True
    else
        echo -e "  - No reboot pending ..."
        REBOOT_REQ=False
    fi
```
This block checks if the file `/var/run/reboot-required` exists. This is the mechanism debian based systems use to reflect a reboot is required for the updates to become fully active.

This block of code can also be used to check if your remote systems already had a reboot requirement pending. I leave that up to you for further exploration.

```bash
    echo -e "\n- Get the FQDN of the host ..."
    REMOTE_NAME=$(ssh ${ADMIN}@$i dig +short -x $i)
```
On the remote host the fully qualified domain name of the host is collected. DNS is not a service that is guaranteed to be implemented free of faults and issues. Asking the host itself for it's name, increases the chance to get a name.

```bash
    # combine logs
    fnCombineLogs ${i} ${REBOOT_REQ} ${REMOTE_NAME}
```
Here the the function `fnCombineLogs` is called to add the log of this host to the output that is already collected. It provides the details of:
* The IP address or resolvable name of the host that has been checked
* If a reboot is required for this host
* The name that the host thinks it has.

```bash
done
```
End of the loop...

```bash
# The End ...
echo -e "\nFinished processing $0\n\n"
```
Output of finishing the script


## GitHub

Find it on GitHub:  
[https://github.com/CrossCloudGuru/DebianListUpdates](https://github.com/CrossCloudGuru/DebianListUpdates)
