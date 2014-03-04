This document has been adapted from http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html

# Setup CKAN from source in a virtual environment
Continuing here assumes that you have a virtual image running a Linux environement (the preferred OS to host CKAN) into which you will setup CKAN from source. Any virtualisation environment will do, but this tutorial makes most sense if you have an environment as described in [serversetup.md](./serversetup.md)

## Create a virtual environment overlay file (optional)
Many virtualisation technologies support the idea of overlay files. An overlay file only stores changes relative to an otherwise hopefully clean base image. QEMU supports overlays and this will create one with

    qemu-img create -b ubuntu-minimal-13.10.img -f qcow2 ckan2.2.ovl

Overlays have the advantage that once you are stuck in a problem because you have done something wrong during setup and you do not know how to fix that, you can simply delete the (small) overlay and start anew by simply re-creating the overlay as described above. If you have a perfectly working system you may also incorporate changes made to the overlay back into the maim image.

For more information on (QEMU) overlays see [the Arch Wiki](https://wiki.archlinux.org/index.php/QEMU#Overlay_storage_images).

Starting from here the ckan2.2.ovl QEMU image is assumed to be used when the virtual operating system environment is started. If you do not want to work with overlays you have to substitute occurrences of `ckan2.2.ovl` with `ubuntu-minimal-13.10.img`

## Start the virtual operating system environment

	qemu-system-x86_64 -machine accel=kvm ckan2.2.ovl

and log into at the system prompt. Afterwards switch to administrative privileges `sudo -i`, providing the same password you used to log into the image, as more prerequisites for CKAN have to be installed. This pre-requisites hold true for Ubuntu 13.10 and CKAN 2.2.

    <a name="requirements" />
## Install required packages
From now on further descriptions rely on the [CKAN provided documentation](http://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html#install-the-required-packages), only deviating in those rare cases when adaptions were required to account for the virtualized server environment.

	apt-get install apt-get install python-dev postgresql libpq-dev python-pip python-virtualenv git-core solr-jetty openjdk-6-jdk wget

Afterwards stop your virtual server. The installation keeps back some steps which will only be executed once you freshly reboot. Still having administrative rights (you notice the '#' on the command line accepting input) issue

    shutdown -h now

## Install CKAN - preparations
Restart you virtual operating system environment

	qemu-system-x86_64 -machine accel=kvm ckan2.2.ovl

log in with our usual credentials and issue

   	sudo mkdir -p /usr/lib/ckan/default
	sudo chown `whoami` /usr/lib/ckan/default

You will be prompted for your password. Notice the back ticks in the second command. Next issue the two commands

   	virtualenv --no-site-packages /usr/lib/ckan/default
    . /usr/lib/ckan/default/bin/activate

Again, notice the . and following space in the second command. The following commands assume that you stay in the python virtual environment (which is different from the operating system virtual environment which is provided by the QEMU virtualisation engine). A `(default)...` prepending your login prompt is indicating that you are in the Python virtualenv. Whenever you have to interrupt one of the following steps, you must re-enter the Python virtual environment by issuing

  	. /usr/lib/ckan/default/bin/activate

at the command prompt.

## Fetching a stable CKAN version
Within the Python virtual environment execute in order

   	pip install -e 'git+https://github.com/ckan/ckan.git@ckan-2.2#egg=ckan'
    pip install -r /usr/lib/ckan/default/src/ckan/requirements.txt

The first command uses `git`to fetch CKAN sources of stable version 2.2, the second command will fetch all dependencies of CKAN itself. You will see some warning messages and your local C compiler compiling native drivers to communicate eg. with the Postgresql Database which will be set up in the next step. Some warning messages are normal and no sign of problems. Afterwards execute

   	deactivate
	. /usr/lib/ckan/default/bin/activate

to assure all changes get written to the virtual environment.

<a name="dbinit" />
## Initialize the database server
CKAN uses [Postgresql](http://www.postgresql.org/) to store its metadata. Version 7.4 onwards is required. Postgresql was installed already in step [Install required packages](#requirements). Execute

   	sudo -u postgres psql -l

and check that a database named `template0` is UTF8 encoded. If not, change that before continuing by [following this blog post](https://secure.m2osw.com/postgresql_change_encoding). Next, create a new database user (and password) for CKAN

   	sudo -u postgres createuser -S -D -R -P ckan_default

and an empty database which will be filled later with CKAN's data tables:

   	sudo -u postgres createdb -O ckan_default ckan_default -E utf-8

## Create a CKAN configuration
By convention configuration files are kept in folder `/etc` so create the necessary directories:

   	sudo mkdir -p /etc/ckan/default
	sudo chown -R `whoami` /etc/ckan/

CKAN's [paster tool](http://docs.ckan.org/en/ckan-2.2/paster.html) assists to create a default configuration in that directory:

   	cd /usr/lib/ckan/default/src/ckan
	paster make-config ckan /etc/ckan/default/development.ini

Next you have to edit `development.ini` and configure our database connection. You can use any editor, but as you are likely in a console window, a good choice is vim:

   	vim /etc/ckan/default/development.ini

Use the arrow keys to navigate to a line starting with _sqlalchemy.url_ and change it as following

   	sqlalchemy.url = postgresql://ckan_default:pass@localhost/ckan_default

where you have to substitute `pass` with the password you assigned to your postgresql user in step [Initialize the database server](#dbinit) and possibly also username (after postgresql://, here ckan_default) as well as the dabase name (after localhost/, here as well ckan_default). Also find the line containing `site_id = ` and change default to something remotely meaningful such as `MyACMEInstance`.

> If you are new to vim: Use the arrow keys up / down to find the line and move to the location where you want to make changes. When you are at the right position print `i` (lowercase I), now you can use `DEL` key to delete stuff and your regular keys to insert new password. When you are done editing, hit the `ESC` key and press `:wq` followed by enter. That should do the changes and bring you back to the command prompt.

## Setting up solr - Take 1
[Solr](http://lucene.apache.org/solr/) is a very powerful search engine powered by JAVA. After a fresh install it is not running as a service by default and requires some hand-tailoring. Edit the file `/etc/default/jetty` eg. using

   	sudo vim /etc/default/jetty

and change the lines containing `NO_START`, `JETTY_HOST` and `JETTY_PORT` to look like this:

	NO_START=0
	JETTY_HOST=0.0.0.0
	JETTY_PORT=8983

Remember to press `i` before you can make changes to the files content and to press `ESC` followed by `:wq` + `Enter` to save changes and get out of the vim editor.

> Specifying `JETTY_HOST=0.0.0.0` means that Jetty and thus solr will accept connections from every client on the Internet. You want do that on a production server! However, as CKAN is installed in a virtual environment, it should be possible to access solr from the outside world, at least for testing purposes.
    
Shutdown your virtual server with

	sudo shutdown -h now

### Intermission: process and network isolation
In order to reach services from your guest OS, you have to specify additional command line parameters for QEMU. To see solr's welcome screen, you have to start QEMU like

   	qemu-system-x86_64 -machine accel=kvm -net nic -net user,hostfwd=tcp:127.0.0.1:5555-:8983 ckan2.2.ovl

If you open a browser in your guest OS and browse to localhost:5555/solr, you should see solr's welcome screen.

> QEMU command line parameters are a moving target. If you get error messages, read `man qemu` or [online](http://wiki.qemu.org/Documentation/Networking)
    
To continue configuring CKAN you have to login again and start Pythons virtual environment with

   	. /usr/lib/ckan/default/bin/activate

> Note that solr came up automatically by setting `NO_START=0`
    
## Setting up solr - Take 2
Next you have to replace the default solr `schema.xml` file with a symlink to the CKAN schema file included in the sources:

	sudo mv /etc/solr/conf/schema.xml /etc/solr/conf/schema.xml.bak
	sudo ln -s /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml /etc/solr/conf/schema.xml

Restart solr so that changes take effect

	sudo service jetty restart

> Normally solr is likely to be set up in a multi-server environment. The CKAN documentation provides help for that so that other services can take advantage from solr. For ease and brevity we will further assume that this instance of solr belongs exclusively to CKAN. If this is not the case, [Multicore Solr Setup](http://docs.ckan.org/en/latest/maintaining/solr-multicore.html) will give you advice how to organize the setup.

## Set up the database
Postgresql is already up and running but the database contains no tables yet. In the next step you will create the database tables:

	cd /usr/lib/ckan/default/src/ckan
	paster db init -c /etc/ckan/default/development.ini

Remember, that you have to execute this command in the virtual Python environment. If you receive a `paster: command not found`, start the environment with

	. /usr/lib/ckan/default/bin/activate

If all succeeds you will be prompted with a `Initialising DB: SUCCESS` message.

## Almost done
CKAN will be served by [Zope ](http://www.zope.org/), a web application server written in Python. Zope requires a `who.ini`-file to be accessible in the same directory as the CKAN configuration file. Establish that link with

	ln -s /usr/lib/ckan/default/src/ckan/who.ini /etc/ckan/default/who.ini

In the next step we will start CKAN which listens by default on port 5000. Once again, in order to be accessible from outside the virtual environment we have to tell QEMU to route tcp packages into our virtual server instance. For that shutdown the virtual engine

	sudo shutdown -h now
    
and restart it with this command line

	qemu-system-x86_64 -machine accel=kvm -net nic -net user,hostfwd=tcp:127.0.0.1:8080-:5000 ckan2.2.ovl
    
Login, create the Python virtual environment `. /usr/lib/ckan/default/bin/activate` and start CKAN

	cd /usr/lib/ckan/default/src/ckan
	paster serve /etc/ckan/default/development.ini

In your host operating system open a web browser and surf to localhost:8080. Congratulations, CKAN is up and running!

In order to stop CKAN press `CTRL-ALT-g`,  `CTRL-c`,  `CTRL-ALT-g`. The key sequence `CTRL-ALT-g` signals the QEMU virtual machine to pass through `special` key sequences.`CTRL-c` is such a special key sequence and will stop the CKAN server. The QEMU menu bar will tell you wheather your virtual machine is capturing special key-codes (default) or if they are passed through the running process.

## Next steps

* Manage your newly created CKAN instance: TODO: Mention here what to do
* Install the datastore extension http://docs.ckan.org/en/latest/maintaining/datastore.html. When you hit the section on installing the DataPusher service, continue with
* Install the [DataPusher](./datapushersetup.md) service
* Server CKAN using a full-featured web server. Think about scaling, load-balancing and fail-over.
* Tune your PostgresSQL server. Postgres comes by default with very defensive performance settings. You should definitely tweak them as described here https://wiki.postgresql.org/wiki/Tuning_Your_PostgreSQL_Server or for an overview https://wiki.postgresql.org/wiki/Performance_Optimization.
