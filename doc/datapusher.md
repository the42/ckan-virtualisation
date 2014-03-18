This file describes setting up the datapusher service http://docs.ckan.org/projects/datapusher/en/latest/. It is based on the original documentation but installs the service in a Python virtual environment which eases deployment and increases flexibility when going from testing to production environment.

# Create a Python virtual environment
It is very likely that you came here after [installing CKAN from source](./ckansetup.md). In the following step we will create a Python virtual environemnt. This assumes that you installed the required [pre-requisites for CKAN](./ckansetup.md).

	sudo mkdir -p /usr/lib/ckan/datapusher
    sudo chown `whoami` /usr/lib/ckan/datapusher
    
    virtualenv --no-site-packages /usr/lib/ckan/datapusher
	. /usr/lib/ckan/datapusher/bin/activate

# Fetch the latest version of datapusher
First we will create a source directory which will hold the datapusher source code

    mkdir -p /usr/lib/ckan/datapusher/src
    cd /usr/lib/ckan/datapusher/src

Next fetch the source code from github. Here we fetch only the stable branch of datapusher:

	git clone -b stable https://github.com/ckan/datapusher.git
	cd datapusher

and install the requirements with

	pip install -r requirements.txt
	pip install -e .

# Configure CKAN to use the datapusher
Open your `/etc/ckan/default/development.ini`-file and find the line `ckan.plugins = ` Whatever plugins are already configured, add (or check if it already contains) ` datapusher` to the list of plugins.

# Start the datapusher
by running

	JOB_CONFIG=/usr/lib/ckan/datapusher/src/datapusher/deployment/datapusher_settings.py python wsgi.py
    
(2014-03-18: Setting the environment variable may be irrelevant at a later time; it is a bug that it get's not set automatically by the `pip setup`). If you logged out and come back at a later time, you have to enable the Python virtual environment with `. /usr/lib/ckan/datapusher/bin/activate` before you cd into `/usr/lib/ckan/datapusher/src/datapusher` and start the datapusher service.

Also note that starting the CKAN server will NOT automatically start the datapusher - you have to start it when you plan to use that plugin from within CKAN.