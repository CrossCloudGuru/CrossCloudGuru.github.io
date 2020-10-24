---
title: "[3/3] Arch Linux in VMware Workstation - The scripts in detail"
date: 2019-12-24T15:15:00-04:00
toc: true
categories:
  - blog
  - tutorial
tags:
  - linux
  - vmware
---

This article is part of the series Arch Linux in a VMware Workstation.

In this third article of the series I will explain part by part the script I have created so you get an understanding how/why I did the things I did. Afterwards you can utilize and modify the script as you see fit.

I assume you have worked through the articles: Article 1: Setup the virtual machine and Article 2: Prepare and execute the scripts and therefore have access to the scripts already.

Now lets get into the details! For the impatient, the link to the full script is at the bottom of this post...

### ArchInstallation.sh - Header

```bash
#!/usr/bin/env bash 
################################################################################ 
# Filename: ArchInstallation.sh
# Date Created: 21-dec-19
# Date last update: 21-dec-19
# Author: Marco Tijbout
#
# Version 0.1
#
# Enhancement ideas:
# -
#
# Version history:
# 0.1 Marco Tijbout:
# Initial release of the script.
################################################################################
```

The header area is for me where I do my housekeeping. An area where I can note what has changed, what my plans are that become future changes/updates.

Every script used should have a line to tell the system what engine / language to use:

```bash
#!/usr/bin/env bash
```

Usually you see `#!/bin/bash` but I have learned to use my format that makes the system look for bash itself instead of assuming its location. So I use `#!/usr/bin/env bash`

When reading the installation guide on the ArchWiki you will notice some sections like pre-installation and installation. I tried to group the items to the sections.

### The Pre-Installation section

```bash
################################################################################
## PRE-INSTALLATION
################################################################################

# Configure to use NTP (preferably already before the script is run):
echo -e "\n * Configure system to use NTP ...\n"
timedatectl set-ntp true

# Check if the installation disk of 40GB is found:
echo -e "\n * Probe the installation disk ...\n"
DISK_NAME=$(fdisk -l | grep "GiB" | awk -F ':' '{print $1}' | awk '{print $2}')
echo -e "Disk: Recognized as $DISK_NAME"

DISK_SIZE_GB=$(fdisk -l | grep "GiB" | awk '{print $3}')
echo -e "Disk: Size in GB: $DISK_SIZE_GB"

# Create a partition layout file for a 40GB disk:
echo -e "\n * Create partition layout file ...\n"
source diskSectorCalculator.sh $DISK_SIZE_GB

# Partition the disk based on the layout file:
echo -e "\n * Partition the disk ...\n"
sfdisk --force $DISK_NAME < layout.sfdisk

# Formatting the freshly created partitions:
echo -e "\n * Formatting the partitions on the disk ...\n"
mkfs.fat -F32 ${DISK_NAME}1
mkfs.ext4 -F ${DISK_NAME}2

# Prep and enable the swap partition:
echo -e "\n * Enable swap partition ...\n"
mkswap ${DISK_NAME}3
swapon ${DISK_NAME}3

# Mounting the partitions:
echo -e "\n * Mounting of the partitions ...\n"
mount ${DISK_NAME}2 /mnt
mkdir -p /mnt/boot
mount ${DISK_NAME}1 /mnt/boot
```

If not already done, synchronize the time with NTP:

```bash
# Configure to use NTP (preferably already before the script is run):
echo -e "\n * Configure system to use NTP ...\n"
timedatectl set-ntp true
```

During boot the Live system identified the disk. For creating a partition scheme we need to know the name it was assigned:

```bash
# Check if the installation disk of 40GB is found:
echo -e "\n * Probe the installation disk ...\n"
DISK_NAME=$(fdisk -l | grep "GiB" | awk -F ':' '{print $1}' | awk '{print $2}')
echo -e "Disk: Recognized as $DISK_NAME"
```

The script assumes that only one disk is attached to the virtual machine and is at least a couple of GB in size.

What happens here is:

* We use information provided by the command `fdisk -l`
* grep GiB filters out the line of information we are looking for
* `awk -F ':' '{print $1}'` We need to cut the required information separating the : from the information we need and use the FIRST part
* `awk '{print $2}'` From this 'first' part we need the second value
* To store it in a variable for later usage we need to wrap it in `$()`

As this script will create a partition layout automatically based on size, we need to figure out the size of the disk:

```bash
# Probe the size of the disk:
DISK_SIZE_GB=$(fdisk -l | grep "GiB" | awk '{print $3}')
echo -e "Disk: Size in GB: $DISK_SIZE_GB"
```

What happens here is:

* We use information provided by the command `fdisk -l`
* grep GiB filters out the line of information we are looking for
* `awk '{print $3}'` From this first part we need the third value

With the information we have collected so far we can create a configuration file that is used to create partition later:

```bash
# Create a partition layout file for the disk:
echo -e "\n * Create partition layout file ...\n"
source diskSectorCalculator.sh $DISK_SIZE_GB
```

What happens here is:

- An external script is called: `diskSectorCalculator.sh`
- By calling it through source all variables in this and the called script will be available. It becomes an integrated part of the script.
- The variable `$DISK_SIZE_GB` at the end will feed its information into the sourced script as input.

NOTE: The inner-works of diskSectorCalculator.sh is discussed further down this article.

Now the partition layout file is calculated and created as a result of the diskSectorCalculator.sh script, it is used as input:

```bash
# Partition the disk based on the layout file:
echo -e "\n * Partition the disk ...\n"
sfdisk --force $DISK_NAME < layout.sfdisk
```

What happens here is:

* `sfdisk` is the tool used to create the partitions
* `--force` tells `sfdisk` to override and answer any objections with YES
* `< layout.sfdisk` reads the content of `layout.sfdisk` as input to the command. Basically applying a template

After the partitions are created, they need to be formatted and the swap partition enabled:

```bash
# Formatting the freshly created partitions:
echo -e "\n * Formatting the partitions on the disk ...\n"
mkfs.fat -F32 ${DISK_NAME}1
mkfs.ext4 -F ${DISK_NAME}2
```

What happens here is:

* `mkfs.fat -F32 ${DISK_NAME}1`
    * `mkfs` is the command to format a partition
    * `.fat` and `-F32` specifies it to be of type FAT32
    * `${DISK_NAME}1` specifies the name of the disk identified by the live system like `/dev/sda` and the addition `1` specifies the first partition
* `mkfs.ext4 -F ${DISK_NAME}2`
    * The same approach as the previous one where .ext4 specifies it to format as a Linux partition
    * `${DISK_NAME}2` specifies the name of the disk identified by the live system like `/dev/sda` and the addition `2` specifies the second partition

```bash
# Prep and enable the swap partition:
echo -e "\n * Enable swap partition ...\n"
mkswap ${DISK_NAME}3
swapon ${DISK_NAME}3
```

What happens here is:

* `mkswap` is the command to create a swap location for the system (kernel)
* `${DISK_NAME}3` specifies the name of the disk identified by the live system like `/dev/sda` and the addition `3` specifies the third partition
* `swapon` is the command to enable the swap location

Now that all partitions are created and formatted we need to mount them to use them to get some content on them:

```bash
# Mounting the partitions:
echo -e "\n * Mounting of the partitions ...\n"
mount ${DISK_NAME}2 /mnt
mkdir -p /mnt/boot
mount ${DISK_NAME}1 /mnt/boot
```

What happens here is:

* `mount` is the command to mount a partition where
    * `${DISK_NAME}2` tells it to use the second partition `/dev/sda2` and
    * `/mnt` specifies the mount point, e.g. the folder to access the partition from
* `mkdir` is the command to create a new directory where
    * `-p` specifies to create the full path of directories. (Is in this case obsolete as `/mnt` already exist. Good habits die hard)
    * `/mnt/boot` specifies the folder boot to be created under `/mnt`

### The installation section:

The target disk is now fully prepared and accessible to put the new system on to it.

To speedup the downloads it is recommendable to use a repository mirror closer to where you are located. I live in the Netherlands and therefore it is beneficial to download from locally instead of trans-Atlantic from the USA.

To figure out the fastest mirror close to you:
Arch Linux – Mirror Status
[https://www.archlinux.org/mirrors/status/](https://www.archlinux.org/mirrors/status)

Or generate a mirror list by country (not used in the script):
Arch Linux – Pacman Mirrorlist Generator
https://www.archlinux.org/mirrorlist/?country

Let\’s make a backup of the original mirror list first:

```bash
# Backup:
echo -e \n * Backup mirrorlist ...\n
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.org
```

Adding the fastest local mirror above the list of available mirrors:

```bash
# Add the mirror information to the top:
echo -e \n * Add Netherlands miror to top of mirrorlist ...\n
sed -i '6i## Netherlands\' /etc/pacman.d/mirrorlist
sed -i '7iServer = ftp.nluug.nl/os/Linux/distr/archlinux/$repo/os/$arch\' /etc/pacman.d/mirrorlist
```

What happens here is:

- With `sed -i` we are able to
  - `-i` specifies that the file needs to be edited. Not only generate alterations as output.
  - The whole command to execute is captured in `' '`
  - In the command `6i` specifies to insert on the 6th row
  - The `\` specifies to add a new line
  - Same for the next command

Now it is time to install Arch Linux onto the new disk:

```bash
# Install essential packages:
echo -e \n * Install essential packages ...\n
pacstrap /mnt base base-devel linux linux-firmware nano
```

What happens here is:
- `pacstrap`  is the command to install the base Arch system
  - `/mnt` specifies where to install to
  - followed by `base base-devel linux linux-firmware nano` that are the system packages and nano as editor

Now the target disk has the basic folder layout and files. When a Linux system boots, the kernel needs to know what partitions to mount:

```bash
# Create FSTAB file
echo -e \n * Create fstab file ...\n
genfstab -U /mnt >> /mnt/etc/fstab
```

What happens here is:
- `genfstab` is the command to generate a file system table file where
  - `-U` tells to use UUIDs for source identifiers
  - `/mnt` as the mount point to start from
  - `/mnt/etc/fstab` where to write the output file

#### Continued installation in chroot:

Now let’s work on this interesting section.

We have a filled new target system but still some actions need to be done in the context of the new system. For that we use a concept like chroot. Wit this command we create a new confined user space that starts from the new root location.

If you want to read more on chroot:
chroot – Wikipedia
https://en.wikipedia.org/wiki/Chroot

What this section does is:
- We create and fill a secondary script ( `ArchInstallation2.sh` ) from within this script with the command `cat`
  - We will discuss the the contents of `ArchInstallation2.sh`
- Make the secondary script executable
- We execute the script in the chroot-ed context with arch-chroot

```bash
################################################################################
## CHROOT - INSTALLATION
################################################################################

# Prepare installation steps during chroot:
echo -e "\n * Prep second stage in chroot ...\n"

# Create the file to be executed the chroot environment:
cat <<EOF > /mnt/root/ArchInstallation2.sh
#!/usr/bin/env bash
################################################################################
# Filename: ArchInstallation2.sh
# Date Created: 21-dec-19
# Date last update: 21-dec-19
# Author: Marco Tijbout
#
# Version 0.1
#
# Enhancement ideas:
#   - Using flags to specify values to arguments.
#   - Option to update the script (-u --update -au (allways update))
#
# Version history:
# 0.1  Marco Tijbout:
#   Initial release of the script.
################################################################################

################################################################################
## CHROOT - INSTALLATION
################################################################################
TARGET_MACHINE=archvm

# Set the locale for the system:
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen

# Generate the locales:
locale-gen

# Configure what language to use:
echo "LANG=en_US.UTF-8" >> /etc/locale.conf

# Set time zone to Europe Amsterdam:
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

# Set the time standard to UTC using command:
hwclock --systohc --utc

# Set the hostname
echo "$TARGET_MACHINE" >> /etc/hostname

# Fill the hosts file:
cat <<EOI > /etc/hosts
127.0.0.1    localhost
::1          localhost
127.0.1.1    $TARGET_MACHINE.localdomain     $TARGET_MACHINE
EOI

# Install and enable netowrkmanager:
pacman -S --noconfirm networkmanager
systemctl enable NetworkManager

# Install and enable OpenSSH
pacman -S --noconfirm openssh
systemctl enable sshd

# Install the bootloader
pacman -S --noconfirm grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg

echo -e "\n * Set password for root ...\n"
echo -e "theiotcloud\ntheiotcloud" | passwd

# Exit this part of the script
exit
EOF

# Make second installation script executable:
echo -e "\n * Make second installation script executable ...\n"
chmod +x /mnt/root/ArchInstallation2.sh

# Switch to newly installed version and continue installation:
echo -e "\n * Chroot and continue installation ...\n"
arch-chroot /mnt /root/ArchInstallation2.sh

################################################################################
## END OF INSTALLATION
################################################################################

echo -e "\n * Back from chroot ...\n"

# Unmount the partitions:
echo -e "\n * Unmount the partitions ...\n"
umount -R /mnt

# End of the script:
echo -e "\n * Script is finished.\nReady to reboot into the new system ...\n"
```



Let\’s create the second stage of this rocket, the script ArchInstallation2.sh:

```bash
# Create the file to be executed the chroot environment:
cat <<EOF > /mnt/root/ArchInstallation2.sh
```

What happens here is:

- We use the command `cat` we ingest everything that comes after until we reach the string `EOF`

  - and `echo` this content into the second script with `> /mnt/root/ArchInstallation2.sh`where

    - `>` is used to start with an empty file
  - at the location `/mnt/root/ArchInstallation2.sh`

The script to ingest by `cat`:

```bash
#!/usr/bin/env bash
################################################################################
# Filename: ArchInstallation2.sh
# Date Created: 21-dec-19
# Date last update: 21-dec-19
# Author: Marco Tijbout
#
# Version 0.1
#
# Enhancement ideas:
#  - Using flags to specify values to arguments.
#  - Option to update the script (-u --update -au (allways update))
#
# Version history:
# 0.1  Marco Tijbout:
#  Initial release of the script.
################################################################################

################################################################################
## CHROOT - INSTALLATION
################################################################################

TARGET_MACHINE=archvm

# Set the locale for the system:
echo en_US.UTF-8 UTF-8 > /etc/locale.gen

# Generate the locales:
locale-gen

# Configure what language to use:
echo LANG=en_US.UTF-8 >> /etc/locale.conf

# Set time zone to Europe Amsterdam:
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

# Set the time standard to UTC using command:
hwclock --systohc --utc

# Set the hostname
echo $TARGET_MACHINE >> /etc/hostname

# Fill the hosts file:
cat <<EOI > /etc/hosts
127.0.0.1    localhost
::1          localhost
127.0.1.1    $TARGET_MACHINE.localdomain    $TARGET_MACHINE
EOI

# Install and enable netowrkmanager:
pacman -S --noconfirm networkmanager
systemctl enable NetworkManager

# Install and enable OpenSSH
pacman -S --noconfirm openssh
systemctl enable sshd

# Install the bootloader
pacman -S --noconfirm grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg

echo -e \n * Set password for root ...\n
echo -e theiotcloud\ntheiotcloud | passwd

# Exit this part of the script
exit
```

Followed by `EOF` to close the ingestion.

Let us dissect this script:

Specify the new name for the Arch Linux VM:

```bash
TARGET_MACHINE=archvm
```

To configure the system locale we have the following 3 commands:

```bash
# Set the locale for the system:
echo en_US.UTF-8 UTF-8 > /etc/locale.gen

# Generate the locales:
locale-gen

# Configure what language to use:
echo LANG=en_US.UTF-8 >> /etc/locale.conf
```

What happens here is:

- We create or overwrite the file `/etc/locale.gen` with the US English locale name `en_US.UTF-8 UTF-8.`
- Based on that information we let the system generate with `locale-gen` the local files to use.
- We need to let the system at boot know what locales to use by adding them to the `locale.conf` file.

A correct system time is important. We specify here the time zone where I am in: (use your own if it differs)

```bash
# Set time zone to Europe Amsterdam:
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

# Set the time standard to UTC using command:
hwclock --systohc --utc
```

What happens here is:

- By creating a symbolic link to the time zone file you can specify the time zone for your machine where
- `ln -sf` is the command to create the link
  - where `-s` specifies that it is of type symbolic
    - where `-f` (both combined to `-sf`) forces to overwrite if already a link exists.
- We use `hwclock --systohc --utc` to have the time synced to the hardware clock and set the clock to UTC

NOTE: You might have the thought now why to sync the virtual machines time to a hardware clock does not make sense, you are correct. At this stage there is no NTP enabled in the new virtual machine and I know that the hwclock command does all kinds of important settings in the background. For now as this is only for the installation, I leave it as is. This is for a separate part to configure the VM for production what is out of scope of this article series.

Next we need to make sure the new VM learns it’s new name:

```bash
# Set the hostname
echo $TARGET_MACHINE >> /etc/hostname

# Fill the hosts file:
cat <<EOI > /etc/hosts
127.0.0.1    localhost
::1          localhost
127.0.1.1    $TARGET_MACHINE.localdomain    $TARGET_MACHINE
EOI
```

What happens here is:

- We echo the name we have specified in the variable `$TARGET_MACHINE` into `/etc/hostname`
- We also need for networking purposes have the system able to resolve it\’s own name
  - Like with the whole script we use the same cat trick to fill the file but…
  - as we are already in a EOF loop we need to specify a different unique name so I chose `EOI` (End Of Information)

As you can see, you can use variables that are defined outside of the loop that will be filled during the processing of this loop. Really handy if you need larger configuration files with an outline and especially multiple lines.

During testing I figured it is very usable to install already two services and enable them:

```bash
# Install and enable networkmanager:
pacman -S --noconfirm networkmanager
systemctl enable NetworkManager

# Install and enable OpenSSH
pacman -S --noconfirm openssh
systemctl enable sshd
```

What happens here is:

- `pacman -S` is the command to install software where
  - `--noconfirm` will suppress confirmation requests
  - `networkmanager` is the package/service to be installed

- Same for `openssh`

The last installation we do is installing the boot loader:

```bash
# Install the bootloader
pacman -S --noconfirm grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

What happens here is:

- `packman -S` is the command to install software where
  - `--noconfirm` will suppress confirmation requests
  - `grub` and `efibootmgr` are the packages/software to be installed.
- `grub-install` to install the grub boot loader where  
  - `--target=x86_64-efi` specifies what type of system it is
  - `--efi-directory=/boot` specifies what location to look for configuration files afterwards
- `grub-mkconfig` is the command to generate the grub config file where
  - `-o /boot/grub/grub.cfg` specifies with `-o` the output file as `/boot/grub/grub.cfg`



Finally I set the root password as I do not like systems without a password for root:

```bash
echo -e \n * Set password for root ...\n
echo -e theiotcloud\ntheiotcloud | passwd
```

What happens here is:

- we `echo -e` twice the password `theiotcloud`

  - separated by the new line carrier return `\n`

  - we use the `-e` option to have `echo` accept specials like the carrier return
  
- and pipe it to the command `passwd` as input



This concludes the script that will be executed in the chroot environment.

Now we have to make it executable:

```bash
echo -e \n * Make second installation script executable ...\n
chmod +x /mnt/root/ArchInstallation2.sh
```

That leaves us to initiate the script in the chroot environment:

```bash
# Switch to newly installed version and continue installation:
echo -e \n * Chroot and continue installation ...\n
arch-chroot /mnt /root/ArchInstallation2.sh
```

What happens here is:

- We initiate the chroot with `arch-root`
- with `/mnt` as starting point e.g. the new root
  - and in the new context execute the script `/root/ArchInstallation2.sh`

Once the script is completed, we are back in the Arch Live environment as the script will exit the chroot environment.



We need gracefully close the new system by unmounting the partitions:

```bash
# Unmount the partitions:
echo -e \n * Unmount the partitions ...\n
umount -R /mnt
```

What happens here is:

- `umount` is the unmount command where
  - `-R` specifies all mounts to unmount under
  - `/mnt` as the start of the tree.



We have now reached the end of the scripts and if executed it leaves you in a system that is ready to reboot into the fresh install.



I hope you liked the articles and learned something new. If you find any errors or know better ways, please share in the comments as well. Navigation:

- [Overview](https://crosscloud.guru/archives/483)
- [Article 1: Setup the virtual machine](https://crosscloud.guru/archives/484)
- [Article 2: Prepare and execute the scripts](https://crosscloud.guru/archives/505)



Here is the link to the full scripts:
[Git repository ArchInstall scripts](https://github.com/CrossCloudGuru/ArchInstall)





