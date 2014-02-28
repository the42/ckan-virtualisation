# Setup CKAN from source in a virtual environment

Continuing here assumes that you have a virtual image runing a linux environemnt (the preferred OS to host CKAN) into which we are going to setup CKAN from source. Any virtualisation environment will do, but this tutorial makes most sense if you have an environment as described in [serversetup.md](./serversetup.md)

0. Create a virtual environment overlay file (optional)

	Many virtualisation technologies support the idea of overlay files. An overlay file only stores changes relativ to an otherwise hopfully clean base image. QEMU, which we are using supports overlays and we will create on with
    	qemu-img create -b ubuntu-minimal-13.10.img -f qcow2 ckan2.2.ovl
    Overlays have the advantage that once you are stuck in a problem because you have done something wrong during setup and you do not know how to fix that, you can simply delete the (small) overlay and start anew by simply re-creating the overlay as described above. If you have a perfectly working system you may also incorporate changes made to the overlay back into the maim image.
    For more information on (QEMU) overlays see [the Arch Wiki](https://wiki.archlinux.org/index.php/QEMU#Overlay_storage_images).
    From now on we will operate on the ckan2.2.ovl QEMU image. If you do not want to work with overlays you have to substitute occurences of `ckan2.2.ovl` with `ubuntu-minimal-13.10.img`

1. Start the virtual operating system environment
		qemu-system-x86_64 -machine accel=kvm ckan2.2.ovl
    and log into at the system prompt. Afterwards switch to administrative privileges `sudo -i`, providing the same password you used to log into the image, as we are going to install some prerequisites for CKAN. This pre-requisites hold true for Ubuntu 13.10 and CKAN 2.2.

2. Install required packages

	From now on we rely on the [CKAN provided documentation](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#install-the-required-packages) 
    	apt-get install apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty openjdk-6-jdk
    Afterwards stop your virtual server. The installation keeps back some steps which will only be executed once you freshly reboot. Still having administrative rights (you notice the '#' on the command line accepting input) issue
    	shutdown -h now

3. Install CKAN from source

	Once again we rely on the excellent [CKAN installation instructions](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#install-ckan-into-a-python-virtual-environment)
    Restart you virtual operating system environemt
		qemu-system-x86_64 -machine accel=kvm ckan2.2.ovl
    log in with our usual credentials and issue
    	sudo mkdir -p /usr/lib/ckan/default
		sudo chown `whoami` /usr/lib/ckan/default
    You will be prompted for your password. Notice the backticks in the second command. Next issue the two commands
    	virtualenv --no-site-packages /usr/lib/ckan/default
        . /usr/lib/ckan/default/bin/activate
    Again, notice the . and following space in the second command. The following commands assume that you stay in the python virtual environemnt (which is different from the operating system virtual environment which is provided by the QEMU virtualisation engine). You notice that you are in the Python virtual environment by prepended `(default)...` at your login prompt.