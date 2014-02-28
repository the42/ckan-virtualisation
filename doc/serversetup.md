# Installing Ubuntu Server as a QEMU Instance

## Install QEMU.
This document assumes that you use at least QEMU version 1.5 onwards. QEMU comes pre-packaged for [x]ubuntu 13.10 in version 1.5 with KVM support. You can install it (assuming an Intel based guest and host system) with
    
    sudo ap-get install qemu-system-x86

## Download Ubuntu server ISO Image.
http://www.ubuntu.com/download/server, preferably 64 bit edition.

## Create a fresh virtual qemu-harddisk which will host the guest-OS
    qemu-img create ubuntu-server-minimal-13.10.img 10G

This will create an image file `ubuntu-server-minimal-13.10.img` with 10 GB virtual disk space. Increase the disk space in case you need more.

## Install the guest OS.
... by booting the ISO-file in the created virtual QEMU-image
    
    qemu-system-x86_64 -enable-kvm -hda ubuntu-server-minimal-13.10.img -cdrom ../Downloads/ubuntu-13.10-server-amd64.iso -boot d -m 2G
	
This will boot into the installer specified by the ISO image ubuntu-13.10-server-amd64.iso mouting it as a CDROM within the virtual environment using the image ubuntu-server-minimal-13.10.img and devoting 2GB of ram

## Install Ubuntu server within the image
You may find additional info at https://help.ubuntu.com/community/Ubuntu_as_Guest_OS. As you are installing in a virtual environment, you are advertised to install a minimal virtual machine by hitting the F4 (modes) key after initial language selection.

When being prompt what software to install, install at least “Minimal Server Packages”. You can install more of the presented server parts anytime at a later point.

After the installation finishes, your minimal Linux server will reboot which will signal QEMU to respawn the process, ie. you will be prompted again with the Ubuntu setup and language selection. Don't fear! Your server is likely properly set up at that stage. Safely terminate it by selecting "Power Down" in the QEMU menu.

## Log into your system
At this stage your server is set up. Start your image with
    
	qemu-system-x86_64 -machine accel=kvm ubuntu-server-minimal-13.10.img
    
If that fails try a more conservative
    
   	qemu ubuntu-server-minimal-13.10.img
    
The later will not give you KVM virtualisation (near bare metal execution speed), expect at least a two-fold slowdown compared to native speed.

## Update packages
Provide your credentials at the login prompt (the username and password you provided during the installation stage). Afterwards issue
    
   	sudo apt-get update
    sudo apt-get upgrade
    
This will update the packages to the latest stable versions, fixing possible security fixes of software packages since your Ubuntu server image file has been provided.
    
Your done!
