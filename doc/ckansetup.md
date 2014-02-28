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

    <a name="requirements" />
2. Install required packages

	From now on we rely on the [CKAN provided documentation](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#install-the-required-packages) 
    	apt-get install apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty openjdk-6-jdk
    Afterwards stop your virtual server. The installation keeps back some steps which will only be executed once you freshly reboot. Still having administrative rights (you notice the '#' on the command line accepting input) issue
    	shutdown -h now

3. Install CKAN - preparations

	Once again we rely on the excellent [CKAN installation instructions](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#install-ckan-into-a-python-virtual-environment)
    Restart you virtual operating system environemt
		qemu-system-x86_64 -machine accel=kvm ckan2.2.ovl
    log in with our usual credentials and issue
    	sudo mkdir -p /usr/lib/ckan/default
		sudo chown `whoami` /usr/lib/ckan/default
    You will be prompted for your password. Notice the backticks in the second command. Next issue the two commands
    	virtualenv --no-site-packages /usr/lib/ckan/default
        . /usr/lib/ckan/default/bin/activate
    Again, notice the . and following space in the second command. The following commands assume that you stay in the python virtual environemnt (which is different from the operating system virtual environment which is provided by the QEMU virtualisation engine). The `(default)...` prepended at your login prompt will indicate that you are in the Python virtualenv. Whenever you have to interrupt the following steps, you can enter the Python virtual env by issuing
    	. /usr/lib/ckan/default/bin/activate
    at the command prompt.

4. Fetching a stable CKAN version

	Within the Python virtual environment execute in order
    	pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.2#egg=ckan'
        pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt
    The first command uses `git`to fetch CKAN sources of stable version 2.2, the second command will fetch all dependencies of CKAN itself. You will see some warning messages and your local C compiler compiling native drivers to communicate eg. with the Postgresql Database we are going to set up un the next step. Some warning messages are normal and no sign of problems. Afterwards execute
    	deactivate
		. /usr/lib/ckan/default/bin/activate
    to assure all changes get written to the virtual environment.

    <a name="dbinit" />
5. Initialize the database server.

	CKAN uses [Postgresql](http://www.postgresql.org/) to store its metadata. Version 7.4 onwards is required. Postgresql was installed already in step [Install required packages](#requirements). Execute
    	sudo -u postgres psql -l
    and check that a database named `template0` is UTF8 encoded. If not, change that before continuing by [following this blog post](https://secure.m2osw.com/postgresql_change_encoding). Next, create a new database user (and password) for CKAN
    	sudo -u postgres createuser -S -D -R -P ckan_default
    and and empty database we will fill later with CKAN's data tables:
    	sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

6. Create a CKAN configuration

	By convention configuration files are kept in folder `/etc` so create the necessary directories:
    	sudo mkdir -p /etc/ckan/default
		sudo chown -R `whoami` /etc/ckan/
    We will use CKAN's [paster tool](http://docs.ckan.org/en/ckan-2.2/paster.html) do fill that directory with a default configuration:
    	cd /usr/lib/ckan/default/src/ckan
		paster make-config ckan /etc/ckan/default/development.ini
    We have to edit `development.ini` and configure our database connection. You can use any editor, but as you are likely in a console window, a good choicce is vim:
    	vim /etc/ckan/default/development.ini
    Use your arrow keys to navigate to a line starting with _sqlalchemy.url_ and change it as following
    	sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default
    where you have to substitute `pass` with the password you assigned to your postgresql user in step [Initialize the database server](#dbinit) and possibly also username (after postgresql://, here ckan_default) as well as the dabase name (after localhost/, here as well ckan_default). Also find the line containing `site_id = ` and change default to something remotely meaningful such as `MyACMEInstance`.
    > If you are new to vim: Use the arrow keys up / down to find the line and move to the location where you want to make changes. When you are at the right position print `i` (lowercase I), now you can use `DEL` key to delete stuff and your regular keys to insert new password. When you are done editing, hit the `ESC` key and press `:wq` followerd by enter. That should do the changes and bring you back to the command prompt.

7. Setting up solr

	[Solr](http://lucene.apache.org/solr/) is a very powerful search engine powered by JAVA. After a fresh install it is not running as a service by default and requires some hand-tailoring. Edit the file `/etc/default/jetty` eg. using
    	sudo vim /etc/default/jetty
    and change the lines containing `NO_START`, `JETTY_HOST` and `JETTY_PORT` to look like this:
    	NO_START=0
		JETTY_HOST=127.0.0.1
		JETTY_PORT=8983
    Remember to press `i` before you can make changes to the content and to press `ESC` followed by `:wq` + `Enter` to get out of the vim editor. Afterwards start the jetty server by issuing
    	sudo service jetty start
    at the command prompt.

DESCRIBE HERE SETTING UP NAT/BEIDGING required because of virtualisation.