THIS second article in the series is about preparing the server side for each of the Raspberry Pi's that will boot over the network. The architecture principle that I have defined is that each Raspberry Pi will get it's own unique and isolated configuration.

This article will describe in detail the build-up of the script I have created during my testing to make it work. A scripted approach helps me automate the steps required to create a repeatable solution. Going over all the steps in the script will provide you a full understanding of my reasoning behind these steps to.

Here is a list of articles related to this project in order to achieve a working environment:

1. Preparing the server - [Setup PXE and NFS on PiServer](https://crosscloud.guru/archives/1124 "Setup PXE and NFS on PiServer")

2. Preparing the guest OS configurations - Prepare netboot OS environment for Raspberry Pi’s
3. Enable the Raspberry Pi's for booting over the network - (under construction)

#### Requirements

This scripted approach requires 2 files:

* `nodeList` - A configuration file that contains information for each Raspberry Pi that you want to enable for booting over the network. Information is stored per line for each Pi with the column order of: `serialnumber` `nodename`
* `prepWorkerImage.sh` - The script that creates the separated Raspberry Pi OS configuration environments based on the input consumed from the file `nodeList`

#### Prepare the nodeList file

The content of the nodeList file has columned information. The current script will read each line and take only into account the first two columns.

* Column 1: Contains the serial number of the Raspberry Pi that it uses to identify itself for unique configuration during the PXE boot to get from the tftp share.
  * Note: The prepHost.sh script adds a configuration that the PXE environment also accepts a MAC address as unique identifier. See article: Setup PXE and NFS on PiServer
  * The MAC address should be denoted with hyphens as: 11-22-33-AA-BB-CC
* Column 2: Contains the hostname to be assigned to the specific Raspberry Pi configuration

An example of a nodeList file could look lite this:

```bash
5d24fd10    pinode01
c9b08141    pinode02
dc-a6-32-75-c8-1c    pinode03
```

#### Explaining the script

The script contains a list of steps in a logical order to enable a separated configuration for a Raspberry Pi to boot from. These steps are wrapped in a function that is called and enriched with the information from an input file containing unique information for each Raspberry Pi. This is preceded by some preparation work not unique for each system to configure.

Now let us dive in to the script starting from the top. The whole script can be found at the bottom of this article.

##### Step 1. Project directory

To keep things organized I use a project directory `netBoot` where all the files files will downloaded to and consumed from.

```bash
## Create if not exist and enter project directory:
[ ! -d ~/netBoot ] && mkdir -p ~/netBoot
cd ~/netBoot
```

##### Step 2. Import and define configuration information

The script consumes information from the nodelist file we have prepared earlier in this article. At this location I also specify values for variables I use later in the script.

```bash
## Collect some details for a specific Raspberry Pi to work with this configuration:
INPUT_FILE=~/nodeList
KICKSTART_IP=10.16.200.1
```

* INPUT_FILE specifies where the script can find the nodelist file.
* The variable KICKSTART_IP defines the master Raspberry Pi server IP address that hosts the NFS and tftp shares used for the PXE boot mechanism.

##### Step 3. Download latest Raspberry Pi OS image

We need to have an image file of the Raspberry Pi OS file on the Pi server that will be used as source for the different Raspberry Pi nodes. To save time and avoid double downloads and unzip actions I have build in some additional logic:

```bash
## Download and unzip the latest Raspbian Buster Lite image:
# Download only if newer timestamp than local file
curl -RLo latest-buster-lite.zip -z latest-buster-lite.zip 
    https://downloads.raspberrypi.org/raspios_lite_armhf_latest

# Get the name of the file in the ZIP:
FILE_IN_ZIP=$(zipinfo -1 latest-buster-lite.zip)

# Extract only if not already extracted:
if [ ! -f ${FILE_IN_ZIP} ]; then
    unzip latest-buster-lite.zip
fi
```

- The download with curl will check if the timestamp of the file on the server is newer than the one on the local disk.
- Next the name of the (first) file in the zip file is retrieved to compare if ...
- Compare if there is already a file with the same name on disk. If not, unzip the zip file.

Step 4. Read input file and execute
At the bottom of the script you find the part that reads the information from the file and iterates through the lines and calling the function doPrepWorker. You find this after the functions you call because a bash script is read in to memory from beginning to end before it executes.

```bash
## Read the input file and execute
while read line ; do
    set $line
    PI_ID=$1
    PI_NAME=$2
    echo -e "nProcessing node:" ${PI_NAME}
    doPrepWorker ${PI_ID} ${PI_NAME}
done < "${INPUT_FILE}"
```

- The input file is read line by line
- For each line the values as we put them in columns are split out into two named variables
- With `doPrepWorker` `S{PI_ID}` `${PI_NAME}` the function is called and the two named variables are passed along

##### Step 5. Function doPrepWorker()

We jump back to the function doPrepWorker. This function contains basically the actual script to be executed for each unique Raspberry Pi system. The function is repeated for each line in the nodeList file.

###### Step 5.1 Creating unique directories

As we want to have full separation of systems for each system a unique NFS and TFTP directory. These directories are created with the information stored in the variables to enable uniqueness.

```bash
# Create directories for separate client images:
echo -e "nCreate NFS folder for node ..."
sudo mkdir -p /nfs/${PI_ID}
echo -e "nCreate TFTP boot folder for node ..."
sudo mkdir -p /tftpboot/${PI_ID}
```

###### Step 5.2 Mounting the OS image

We need to make the downloaded image accessible to get to it's contents.

```bash
# Mount the Raspberry Pi OS image:
echo -e "nMount the Raspberry Pi OS Image ..."
# Create mount points
[ -d rootmnt ] || mkdir rootmnt
[ -d bootmnt ] || mkdir bootmnt

# Make the image file accessible
sudo kpartx -a -v latest-buster-lite.img

# Mount the partitions in the image to the mountpoints
sudo mount /dev/mapper/loop0p2 rootmnt/
sudo mount /dev/mapper/loop0p1 bootmnt/
```

- First we create mount points for the two partitions in the OS image file
- Next we make the OS image file accessible to the system
- Lastly we mount the partitions to know mount points to get access to the files and folders

###### Step 5.3 Copy contents to node directories

Now that the image file is mounted we need to copy over the contents of both partitions to the two respective folders we uniquely created for each Raspberry Pi system. Once completed, the mounts are un-mounted.

```bash
# Copy the Raspbian Buster Lite image to the network boot client image directory created above:
echo -e "nCopy content from root mount ..."
sudo cp -a rootmnt/* /nfs/${PI_ID}/
echo -e "nCopy content from boot mount ..."
sudo cp -a bootmnt/* /nfs/${PI_ID}/boot/

# Un-mount the mountpoints
echo -e "nDone. Unmounting ..."
sudo umount rootmnt
sudo umount bootmnt
```

###### Step 5.4 Replace some boot files

As we are still dealing with beta functionality we need to replace two other files required for the boot process. We will delete the old ones, replace them with new ones and set appropriate permissions.

```bash
# We need to replace the default rPI firmware files with the latest version by running the following commands:
echo -e "nUpdate some firmware files to enable netboot ..."

# Remove current
sudo rm /nfs/${PI_ID}/boot/start4.elf
sudo rm /nfs/${PI_ID}/boot/fixup4.dat

# Download latest
sudo wget https://github.com/Hexxeh/rpi-firmware/raw/stable/start4.elf -P /nfs/${PI_ID}/boot/
sudo wget https://github.com/Hexxeh/rpi-firmware/raw/stable/fixup4.dat -P /nfs/${PI_ID}/boot/

# Correct permissions
sudo chmod 755 /nfs/${PI_ID}/boot/start4.elf
sudo chmod 755 /nfs/${PI_ID}/boot/fixup4.dat
```

###### Step 5.5 Remove SD-Card partition mount points

While using the boot information from the TFTP share all mount points to the SD-Card partitions needs to be removed.

```bash
# Ensure the network boot client image doesn't attempt to look for filesystems on the SD Card:
echo -e "nRemove any SD Card mount-points ..."
sudo sed -i /UUID/d /nfs/${PI_ID}/etc/fstab
```

The command `sed` in-place edits the file and removes all lines that contain `UUID` in it.

###### Step 5.6 Add NFS share mount point

Now that the mount-points to the SD-Card are removed, we need to add to the boot configuration file `/nfs/${PI_ID}/boot/cmdline.txt` the unique share and folder location for each individual system.

```bash
# Replace the boot command in the network boot client image to boot from a network share.
echo -e "nUpdate cmdline.txt to boot from NFS share ..."
echo "console=serial0,115200 console=tty root=/dev/nfs nfsroot=${KICKSTART_IP}:/nfs/${PI_ID},vers=3 rw ip=dhcp rootwait elevator=deadline modprobe.blacklist=bcm2835_v4l2" | sudo tee /nfs/${PI_ID}/boot/cmdline.txt
```

###### Step 5.7 Enable SSH at boot

By adding an empty file called ssh to the boot folder in the unique NFS share, the new Raspberry Pi will enable SSH access for remote management like with setting up a system for headless operations.

```bash
# Enable SSH in the network boot client image:
echo -e "nEnable SSH at first boot of Raspberry Pi ..."
sudo touch /nfs/${PI_ID}/boot/ssh
```

###### Step 5.8 Add NFS share to the server's configuration

Next we need to add the configuration of the NFS share to the server system to enable the share exposure.

```bash
# Create a network share containing the network boot client image:
echo -e "nCreate NFS share for node ..."
echo "/nfs/${PI_ID} *(rw,sync,no_subtree_check,no_root_squash)" | sudo tee -a /etc/exports
```

###### Step 5.9 Prepare the TrivialFTP (TFTP) folder

Although this step is only required once at the first time to setup a boot environment, It cannot hurt to have it in there to have it automatically updated once a newer Raspberry Pi OS image is used.

```bash
# Create a TrivialFTP folder containing boot code for all network boot clients
echo -e "nCreate TFTP root folder ..."
[ -d /tftpboot ] || mkdir -p /tftpboot
[ -f /tftpboot/bootcode.bin ] || sudo cp /nfs/${PI_ID}/boot/bootcode.bin /tftpboot/bootcode.bin
sudo chmod 777 /tftpboot

# Copy the boot directory from the /nfs/${PI_ID} directory to the new directory 
# in /tftpboot:
echo -e "nCopy boot content to TFTP boot folder ..."
sudo cp -a /nfs/${PI_ID}/boot/* /tftpboot/${PI_ID}/
```

1. Copy over the latest bootcode.bin file to the /tftpboot folder
2. Copy over the contents of the boot folder from the new systems NFS share for the netboot procedure.

How it works: What happens is that a Raspberry Pi receives the information where to find the files required to boot. First thing is that the `bootcode.bin` file is read and executed. Next is that the Raspberry Pi has a mechanism built-in to uniquely identify itself with the last 8 characters of it's serial number as we have specified in the variable `${PI_ID}`. OR if the system is enabled to use MAC addresses for identification, the MAC address is used. The possibility to use MAC addresses is enabled in the server side setup of this project.

###### Step 5.10 Assign hostname to the Raspberry Pi configuration

Each system should have it's own unique name. One of the advantages of using a netboot environment is that the configuration files of the individual systems is already accessible for pre-configuration. So we edit here the /etc/hosts and the /etc/hostname file that contain the details for the systems hostname

```bash
## Update the /etc/hosts file with the new name.
echo -e "nAssigning new computername to the Raspberry Pi image ..."
# Modify hosts file
if grep -Fq "127.0.1.1" /nfs/${PI_ID}/etc/hosts
then
    # If found, replace the line
    sudo sed -i "/127.0.1.1/c127.0.1.1    ${PI_NAME}" /nfs/${PI_ID}/etc/hosts
else
    # If not found, add the line
    echo '127.0.1.1    '${PI_NAME} &>> /nfs/${PI_ID}/etc/hosts
fi

# Modify hostname file
sudo sed -i "/raspberry/c${PI_NAME}" /nfs/${PI_ID}/etc/hostname
```

##### Step 6. Reboot the server

Now that all the environments and configurations are created for the Raspberry Pi's listed in the nodeList file, they are ready to boot over the network. Almost. As we have added some new NFS and TFPT shares to the server configuration, I found it is required to reboot the server as restarting the services does not provide me the desired results. Once rebooted, the Raspberry Pi's can boot over the network without any issues.

```bash
sudo reboot
```

#### The full script

In the below GitHub repository you can find both files discussed in this article
[CrossCloudGuru / netBootRPI repository](https://github.com/CrossCloudGuru/netBootRPI "CrossCloudGuru / netBootRPI repository")

The next article in the series will describe the steps required to enable Raspberry Pi's to boot over the network.

Stay tuned!