# Installing Ubuntu Server as a QEMU Instance

1. Install QEMU.

    This document assumes that you use at least QEMU version 1.5 onwards. QEMU comes pre-packaged for [x]ubuntu 13.10 in version 1.5 with KVM support. You can install it (assuming an Intel based guest and host system) with
        sudo ap-get install qemu-system-x86

2. Download Ubuntu server ISO Image.

    http://www.ubuntu.com/download/server, preferably 64 bit edition.

3. Create a fresh virtual qemu-harddisk which will host the guest-OS
		qemu-img create ubuntu-server-minimal-13.10.img 10G

    This will create an image file  ubuntu-server-minimal-13.10.img with 10 GB virtual disk space. Change the disk space in case you need more

4. Install the guest OS.

    By booting the ISO-file in the created virtual QEMU-image
        qemu-system-x86_64 -enable-kvm -hda ubuntu-server-minimal-13.10.img -cdrom ../Downloads/ubuntu-13.10-server-amd64.iso -boot d -m 2G
	This will boot into the installer specified by the ISO image ubuntu-13.10-server-amd64.iso mouting it as a CDROM within the virtual environment using the image ubuntu-server-minimal-13.10.img and devoting 2GB of ram

5. Install Ubuntu server in the image.

	You may find additional info at https://help.ubuntu.com/community/Ubuntu_as_Guest_OS. As you are installing in a virtual environment, you are advertised to install a minimal virtual machine by hitting the F4 (modes) key after initial language selection.

6. When being prompt what software to install, install at least “Minimal Server Packages”. You can install more of the presented server parts anytime at a later point.

7. After the installation finishes, your minimal linux server will reboot which will signal QEMU to respawn the process, iE you will be prompted again with the Ubuntu setup and language selection. Don't fear! Your server is likely properly set up at that stage. Savely terminate it by selecting "Power Down" in the Qemu menu.

8. Log into your system

	At this stage your server is set up. Start your image with
    	qemu-system-x86_64 -machine accel=kvm ubuntu-server-minimal-13.10.img
    If that failes try a more conservative
    	qemu ubuntu-server-minimal-13.10.img
    The later will not give you KVM virtualisation (near bare metal execution speed), expect at least a two-fold slowdown compared to native speed.

9. Update packages

	Provide your credentials at the login prompt (the username and password you provided during the installation stage). Afterwards issue
    	sudo apt-get update
        sudo apt-get upgrade
    This will update the packages to the latest stable versions, fixing possible security fixes of software packages since your ubuntu server image file has been provided.
    
Your done!
