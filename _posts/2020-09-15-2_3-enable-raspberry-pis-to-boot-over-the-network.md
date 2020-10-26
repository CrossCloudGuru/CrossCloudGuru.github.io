---
title: "[2/3] Enable Raspberry Pi's to boot over the network"
date: 2020-09-15T15:14:00-04:00
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

This thrird article is about enabling the Raspberry Pi to boot over the network. As I wanted to test with both my Raspberry Pi versions 3 and 4, I created a script that detects the type and run the specific required steps for the enablement.

<!--more-->

**DISCLAIMER**: This article discusses the usage of Beta firmware for the Raspberry Pi 4. Use at your own risk.

You could argue if this script is actually the second step in the project as it also collects required information that is required to setup the server side OS environment for this Raspberry Pi the script is executed on.

Also worth knowing is that booting over the network is only possible with Raspberry Pi 3B Plus or 4B. I figured this out the hard way. That is the reason that I build a test in this script to see what type of Pi we are dealing with.

During testing and creating this script it evolved into this multi-parted script. The script has five parts:

* 1 part general stuff - The system needs to be updated to acquire the required latest files.
* 3 parts functions - The parts that do the actual work
* 1 part logic - The selector to what functions need to be executed based on the type of the Raspberry Pi

Let me take you through all the parts in chronological order. Bash scripts are fully read into memory before the actual execution starts. Here we go.

### The script explained in chunks

#### Part 1. General - Updating the system


```bash
## Update and upgrade the rPI 
sudo apt update && sudo apt -y full-upgrade
```

#### Part 2. Function - `collectPIDetails ()`

This function collects information about the system we need and store it in the `hardwareInfo.txt` file. Amongst other details it stores the last 8 characters of the serial number that the Pi uses to uniquely identify itself to the PiServer to get it's configuration.  
As it is also possible to use the MAC address as unique identifier, it converts the MAC address of the ethernet port to the format the TFTP server can consume it. Meaning the : (colons) are replaced with - (hyphens).

```bash
collectPIDetails() {
    ## Collect system information and store in a file
    cat /proc/cpuinfo | grep Model > hardwareInfo.txt
    cat /proc/cpuinfo | grep Serial >> hardwareInfo.txt
    cat /proc/cpuinfo | grep Hardware >> hardwareInfo.txt
    cat /proc/cpuinfo | grep Revision >> hardwareInfo.txt

    MAC_ETH=$(cat /sys/class/net/eth0/address)
    echo "MAC eth0        : $MAC_ETH" >> hardwareInfo.txt

    # Convert the MAC address to a string consumable by the TFTP boot process
    MAC_ETH1="$(tr ":" - <<<$MAC_ETH)"
    echo "MAC eth0 (TFTP) : $MAC_ETH1" >> hardwareInfo.txt

    MAC_WLN=$(cat /sys/class/net/wlan0/address)
    echo "MAC wlan0       : $MAC_WLN" >> hardwareInfo.txt

    SERIALNR=$(cat /proc/cpuinfo | grep Serial)
    echo "Netboot serial  : ${SERIALNR:(-8)}" >> hardwareInfo.txt

    cat hardwareInfo.txt
}
```

#### Part 3. Funcion - `preparePI3Bp()`

This function takes care of the Raspberry Pi 3B Plus systems. What it does is:

* It retrieves the value in the boot mechanism that specifies if `netboot` is enabled or not. The value for when it is enabled is stored in `OPT_NETBOOT`.
* The if statement checks the value from the system stored in `OPT_VALUE` with the value in the `OPT_NETBOOT` to see if it is already enabled for netboot or not.
* If the values match the Raspberry Pi is already enabled for booting over the network. The script exits here.
* If the value does not match an additional line is added to the `/boot/config.txt` to enable the Pi.
* Next the script wil exit and the Raspberry Pi needs to be rebooted to make the actual change.

```bash
preparePI3Bp() {
    ## Configuring the Raspberry Pi 3 Model B Plus for PXE booting

    # Check if Pi is already configured
    OTP_VALUE=$(vcgencmd otp_dump | grep 17:)
    OTP_NETBOOT="17:3020000a"

    if  [[ ${OTP_VALUE} == ${OTP_NETBOOT} ]] ; then
        echo -e "\nThis Raspberry Pi 3B Plus is already Netboot enabled."
        exit 0
    else
        echo -e "\nPrepare the Raspberry Pi 3B Plus to enable Netboot."
        echo program_usb_boot_mode=1 | sudo tee -a /boot/config.txt
        echo -e "\nReboot required. Please reboot the Raspberry Pi"
    fi
}
```

#### Part 4. Function - `preparePI4()`
This function is specific for the Raspberry Pi 4. As booting over the network is in BETA, the BETA firmware needs to be loaded in the EEPROM of the Pi.

For this article I use the beta firmware that is released on 31st of July 2020. This specific version is specified in the PI_EEPROM_VERSION variable.
If you want to check if there are any newer versions, check here: [Online Raspberry Pi BETA firmwares](https://github.com/raspberrypi/rpi-eeprom/tree/master/firmware/beta)


So what happens here is:

* The version of the firmware to use is stored in `PI_EEPROM_VERSION`
* Next this specified version is downloaded from the Raspberry Pi GitHub repository. (After updating the system it is actually also present on the Pi itself)
* Next the configuration information from the firmware is extracted to `bootconf.txt`
* In this extracted configuration the value to enable netboot is changed to `0x21`
* Next a customized firmware package is created with the modified configuration file.
* Finally the modified firmware is presented to the system to be applied at the next boot. A message will display that you need to reboot the Raspberry Pi.

```bash
preparePI4() {
    ## Configuring the Raspberry Pi 4 Model B for PXE booting

    ## Specify latest beta eeprom firmware
    PI_EEPROM_VERSION=pieeprom-2020-07-31

    # Donwload the beta eeprom firmware
    wget https://github.com/raspberrypi/rpi-eeprom/raw/master/firmware/beta/${PI_EEPROM_VERSION}.bin
    sudo rpi-eeprom-config ${PI_EEPROM_VERSION}.bin > bootconf.txt
    sed -i 's/BOOT_ORDER=.*/BOOT_ORDER=0x21/g' bootconf.txt
    sudo rpi-eeprom-config --out ${PI_EEPROM_VERSION}-netboot.bin --config bootconf.txt ${PI_EEPROM_VERSION}.bin
    sudo rpi-eeprom-update -d -f ./${PI_EEPROM_VERSION}-netboot.bin
}
```

#### Part 5. The script's logic

This part is the brains of the script. What it does is:

* Retrieve the model name of the current Raspberry Pi and store it in the variable `CURRENT_PI`
* Store the most likely to use Raspberry Pi model names in separate variables to check against
* Next it calls the function collectPIDetails to collect information about this Pi and store it in the file `hardwareInfo.txt` for later usage.
* The next part checks agains the possible options stored and the retrieved model name. Based on the match, it will call the respective function to prepare the system.

```bash
################################################################################
## Script Logic

## Determine model of Raspberry Pi
CURRENT_PI=$(cat /proc/cpuinfo | grep Model)
#echo $CURRENT_PI
PI_4B="Raspberry Pi 4 Model B"
PI_3Bp="Raspberry Pi 3 Model B Plus"
PI_3B="Raspberry Pi 3 Model B Rev"

# Call function to store information about the Raspberry Pi for later consumption
collectPIDetails

# Select what to do based on type of Raspberry Pi
case $CURRENT_PI in
    *${PI_3B}* )
        echo -e "\nThis is a ${PI_3B}"
        ;;
    *${PI_3Bp}* )
        echo -e "\nThis is a ${PI_3Bp}"
        preparePI3Bp
        ;;
    *${PI_4B}* )
        echo -e "\nThis is a ${PI_4B}"
        preparePI4
        ;;
    * )
        echo -e "Raspberry Pi is not identified";;
esac
```

### Full Script

When all is stitched together you get the full script as shown here:

```bash
#!/usr/bin/env bash
################################################################################
# Script name: prepNetbootWorker.sh
# Version: 1.0
#
# Usage:
#   This script needs to be executed on the Raspberry Pi itself.
################################################################################

##
## Worker Configuration
##

################################################################################
## General operations

## Update and upgrade the rPI
sudo apt update && sudo apt -y full-upgrade

################################################################################
## Script Functions. Goes before the Script Logic

collectPIDetails() {
    ## Collect system information and store in a file
    cat /proc/cpuinfo | grep Model > hardwareInfo.txt
    cat /proc/cpuinfo | grep Serial >> hardwareInfo.txt
    cat /proc/cpuinfo | grep Hardware >> hardwareInfo.txt
    cat /proc/cpuinfo | grep Revision >> hardwareInfo.txt

    MAC_ETH=$(cat /sys/class/net/eth0/address)
    echo "MAC eth0        : $MAC_ETH" >> hardwareInfo.txt

    # Convert the MAC address to a string consumable by the TFTP boot process
    MAC_ETH1="$(tr ":" - <<<$MAC_ETH)"
    echo "MAC eth0 (TFTP) : $MAC_ETH1" >> hardwareInfo.txt

    MAC_WLN=$(cat /sys/class/net/wlan0/address)
    echo "MAC wlan0       : $MAC_WLN" >> hardwareInfo.txt

    SERIALNR=$(cat /proc/cpuinfo | grep Serial)
    echo "Netboot serial  : ${SERIALNR:(-8)}" >> hardwareInfo.txt

    cat hardwareInfo.txt
}

preparePI3Bp() {
    ## Configuring the Raspberry Pi 3 Model B Plus for PXE booting

    # Check if Pi is already configured
    OTP_VALUE=$(vcgencmd otp_dump | grep 17:)
    OTP_NETBOOT="17:3020000a"

    if  [[ ${OTP_VALUE} == ${OTP_NETBOOT} ]] ; then
        echo -e "\nThis Raspberry Pi 3B Plus is already Netboot enabled."
        exit 0
    else
        echo -e "\nPrepare the Raspberry Pi 3B Plus to enable Netboot."
        echo program_usb_boot_mode=1 | sudo tee -a /boot/config.txt
        echo -e "\nReboot required. Please reboot the Raspberry Pi"
    fi
}

preparePI4() {
    ## Configuring the Raspberry Pi 4 Model B for PXE booting

    ## Specify latest beta eeprom firmware
    PI_EEPROM_VERSION=pieeprom-2020-07-31

    # Donwload the beta eeprom firmware
    wget https://github.com/raspberrypi/rpi-eeprom/raw/master/firmware/beta/${PI_EEPROM_VERSION}.bin
    sudo rpi-eeprom-config ${PI_EEPROM_VERSION}.bin > bootconf.txt
    sed -i 's/BOOT_ORDER=.*/BOOT_ORDER=0x21/g' bootconf.txt
    sudo rpi-eeprom-config --out ${PI_EEPROM_VERSION}-netboot.bin --config bootconf.txt ${PI_EEPROM_VERSION}.bin
    sudo rpi-eeprom-update -d -f ./${PI_EEPROM_VERSION}-netboot.bin
}

################################################################################
## Script Logic

## Determine model of Raspberry Pi
CURRENT_PI=$(cat /proc/cpuinfo | grep Model)
#echo $CURRENT_PI
PI_4B="Raspberry Pi 4 Model B"
PI_3Bp="Raspberry Pi 3 Model B Plus"
PI_3B="Raspberry Pi 3 Model B Rev"

# Call function to store information about the Raspberry Pi for later consumption
collectPIDetails

# Select what to do based on type of Raspberry Pi
case $CURRENT_PI in
    *${PI_3B}* )
        echo -e "\nThis is a ${PI_3B}"
        ;;
    *${PI_3Bp}* )
        echo -e "\nThis is a ${PI_3Bp}"
        preparePI3Bp
        ;;
    *${PI_4B}* )
        echo -e "\nThis is a ${PI_4B}"
        preparePI4
        ;;
    * )
        echo -e "Raspberry Pi is not identified";;
esac
```




