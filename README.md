<img alt="WebODM" src="https://user-images.githubusercontent.com/1951843/34074943-8f057c3c-e287-11e7-924d-3ccafa60c43a.png" width="180">

[![Build Status](https://travis-ci.org/OpenDroneMap/WebODM.svg?branch=master)](https://travis-ci.org/OpenDroneMap/WebODM) [![Join Gitter Chat](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/OpenDroneMap/web-development) [![GitHub version](https://badge.fury.io/gh/OpenDroneMap%2FWebODM.svg)](https://badge.fury.io/gh/OpenDroneMap%2FWebODM)

A free, user-friendly, extendable application and [API](http://docs.webodm.org) for drone image processing. Generate georeferenced maps, point clouds, elevation models and textured 3D models from aerial images. It uses [ODM](https://github.com/OpenDroneMap/ODM) for processing.


![image](https://user-images.githubusercontent.com/1951843/33631371-5c55cc2c-d9d8-11e7-8609-b9032d4bbbb6.png)


* [Getting Started](#getting-started)
    * [Add More Processing Nodes](#add-more-processing-nodes)
    * [Enable SSL](#enable-ssl)
    * [Where Are My Files Stored?](#where-are-my-files-stored)
    * [Common Troubleshooting](#common-troubleshooting)
    * [Backup and Restore](#backup-and-restore)
    * [Reset Password](#reset-password)
    * [Manage Plugins](#manage-plugins)
 * [Customizing and Extending](#customizing-and-extending)
 * [API Docs](#api-docs)
 * [ODM, NodeODM, WebODM... what?](#odm-nodeodm-webodm-what)
 * [Roadmap](#roadmap)
 * [Getting Help](#getting-help)
 * [Support the Project](#support-the-project)
 * [Become a Contributor](#become-a-contributor)
 * [Architecture Overview](#architecture-overview)
 * [Run the docker version as a Linux Service](#run-the-docker-version-as-a-linux-service)
 * [Run it natively](#run-it-natively)
 * [Run it on the cloud (Google Compute, Amazon AWS)](#run-it-on-the-cloud-google-compute-amazon-aws)
 

![Alt text](https://user-images.githubusercontent.com/1951843/28586405-af18e8cc-7141-11e7-9853-a7feca7c9c6b.gif)

![Alt text](/screenshots/pointcloud.png?raw=true "3D Display")


## Getting Started

* Install the following applications (if they are not installed already):
 - [Docker](https://www.docker.com/)
 - [Python](https://www.python.org/downloads/)
 - [Pip](https://pypi.python.org/pypi/pip/)
 - [Git](https://git-scm.com/downloads)

* Windows users have a choice between Docker Toolbox (Windows 10 Home or older) and Docker for Windows (Windows 10 Pro or newer). Docker for Windows users should set up their Docker environment before launching WebODM using the Docker utility in the system tray: 1) make sure Linux containers are enabled (Switch to Linux Containers...), 2) give Docker enough CPUs (default 2) and RAM (>4Gb, 16Gb better but leave some for Windows) by going to Settings -- Advanced, and 3) select where on your hard drive you want virtual hard drives to reside (Settings -- Advanced -- Images & Volumes). 

* From the Docker Quickstart Terminal or Powershell (Windows), or from the command line (Mac / Linux), type:
```bash
git clone https://github.com/OpenDroneMap/WebODM --config core.autocrlf=input --depth 1
cd WebODM
./webodm.sh start
```

* Open a Web Browser to `http://localhost:8000` (unless you are on Windows using Docker Toolbox, see below)

Docker Toolbox users need to find the IP of their docker machine by running this command from the Docker Quickstart Terminal:

```bash
docker-machine ip
192.168.1.100 (your output will be different)
```

The address to connect to would then be: `http://192.168.1.100:8000`.

To stop WebODM press CTRL+C or run:

```
./webodm.sh stop
```

To update WebODM to the latest version use:

```bash
./webodm.sh update
```

We recommend that you read the [Docker Documentation](https://docs.docker.com/) to familiarize with the application lifecycle, setup and teardown, or for more advanced uses. Look at the contents of the webodm.sh script to understand what commands are used to launch WebODM.

For Windows and macOS users an [installer](https://www.webodm.org/installer) is also available.

You can also run WebODM from a Live USB/DVD. See [LiveODM](https://www.opendronemap.org/liveodm/).

### Add More Processing Nodes

WebODM can be linked to one or more processing nodes running [NodeODM](https://github.com/OpenDroneMap/NodeODM). The default configuration already includes a "node-odm-1" processing node which runs on the same machine as WebODM, just to help you get started. As you become more familiar with WebODM, you might want to install processing nodes on separate machines.

Adding more processing nodes will allow you to run multiple jobs in parallel. 

You **will not be able to distribute a single job across multiple processing nodes**. We are actively working to bring this feature to reality, but we're not there yet. 

### Enable SSL

WebODM has the ability to automatically request and install a SSL certificate via [Let’s Encrypt](https://letsencrypt.org/), or you can manually specify your own key/certificate pair.

 - Setup your DNS record (webodm.myorg.com --> IP of server).
 - Make sure port 80 and 443 are open.
 - Run the following:

```bash
./webodm.sh restart --ssl --hostname webodm.myorg.com
```

That's it! The certificate will automatically renew when needed.

If you want to specify your own key/certificate pair, simply pass the `--ssl-key` and `--ssl-cert` option to `./webodm.sh`. See `./webodm.sh --help` for more information.

### Where Are My Files Stored?

When using Docker, all processing results are stored in a docker volume and are not available on the host filesystem. If you want to store your files on the host filesystem instead of a docker volume, you need to pass a path via the `--media-dir` option:

```bash
./webodm.sh restart --media-dir /home/user/webodm_data
```

Note that existing task results will not be available after the change. Refer to the [Migrate Data Volumes](https://docs.docker.com/engine/tutorials/dockervolumes/#backup-restore-or-migrate-data-volumes) section of the Docker documentation for information on migrating existing task results.

### Common Troubleshooting

Sympthoms | Possible Solutions
--------- | ------------------
While starting WebODM you get: `from six.moves import _thread as thread ImportError: cannot import name _thread` | Try running: `sudo pip install --ignore-installed six`
While starting WebODM you get: `'WaitNamedPipe','The system cannot find the file specified.'` | 1. Make sure you have enabled VT-x virtualization in the BIOS.<br/>2. Try to downgrade your version of Python to 2.7
While Accessing the WebODM interface you get: `OperationalError at / could not translate host name “db” to address: Name or service not known` or `ProgrammingError at / relation “auth_user” does not exist` | Try restarting your computer, then type: `./webodm.sh restart`
Task output or console shows one of the following:<ul><li>`MemoryError`</li><li>`Killed`</li></ul> |  Make sure that your Docker environment has enough RAM allocated: [MacOS Instructions](http://stackoverflow.com/a/39720010), [Windows Instructions](https://docs.docker.com/docker-for-windows/#advanced)
After an update, you get: `django.contrib.auth.models.DoesNotExist: Permission matching query does not exist.` | Try to remove your WebODM folder and start from a fresh git clone
Task fails with `Process exited with code null`, no task console output - OR - console output shows `Illegal Instruction` - OR - console output shows `Child returned 132` | If the computer running NodeODM is using an old or 32bit CPU, you need to compile [OpenDroneMap](https://github.com/OpenDroneMap/OpenDroneMap) from sources and setup NodeODM natively. You cannot use docker. Docker images work with CPUs with 64-bit extensions, MMX, SSE, SSE2, SSE3 and SSSE3 instruction set support or higher.
On Windows, docker-compose fails with `Failed to execute the script docker-compose` | Make sure you have enabled VT-x virtualization in the BIOS
Cannot access WebODM using Microsoft Edge on Windows 10 | Try to tweak your internet properties according to [these instructions](http://www.hanselman.com/blog/FixedMicrosoftEdgeCantSeeOrOpenVirtualBoxhostedLocalWebSites.aspx)
Getting a `No space left on device` error, but hard drive has enough space left | Docker on Windows by default will allocate only 20GB of space to the default docker-machine. You need to increase that amount. See [this link](http://support.divio.com/local-development/docker/managing-disk-space-in-your-docker-vm) and [this link](https://www.howtogeek.com/124622/how-to-enlarge-a-virtual-machines-disk-in-virtualbox-or-vmware/)
Cannot start WebODM via `./webodm.sh start`, error messages are different at each retry | You could be running out of memory. Make sure you have enough RAM available. 2GB should be the recommended minimum, unless you know what you are doing

Have you had other issues? Please [report them](https://github.com/OpenDroneMap/WebODM/issues/new) so that we can include them in this document.

### Backup and Restore

If you want to move WebODM to another system, you just need to transfer the docker volumes (unless you are storing your files on the file system).

On the old system:

```bash
mkdir -v backup
docker run --rm --volume webodm_dbdata:/temp --volume `pwd`/backup:/backup ubuntu tar cvf /backup/dbdata.tar /temp
docker run --rm --volume webodm_appmedia:/temp --volume `pwd`/backup:/backup ubuntu tar cvf /backup/appmedia.tar /temp
```

Your backup files will be stored in the newly created `backup` directory. Transfer the `backup` directory to the new system, then on the new system:

```bash
ls backup # --> appmedia.tar  dbdata.tar
./webodm.sh start && ./webodm.sh down # Create volumes
docker run --rm --volume webodm_dbdata:/temp --volume `pwd`/backup:/backup ubuntu bash -c "rm -fr /temp/* && tar xvf /backup/dbdata.tar"
docker run --rm --volume webodm_appmedia:/temp --volume `pwd`/backup:/backup ubuntu bash -c "rm -fr /temp/* && tar xvf /backup/appmedia.tar"
./webodm.sh start
```

### Reset Password

If you forgot the password you picked the first time you logged into WebODM, to reset it just type:

```bash
./webodm.sh start && ./webodm.sh resetadminpassword newpass
```

The password will be reset to `newpass`. The command will also tell you what username you chose.

### Manage Plugins

To list all available plugins type:

```bash
./webodm.sh plugin list
```

To enable/disable a plugin type:

```bash
./webodm.sh plugin enable <plugin name>
./webodm.sh plugin disable <plugin name>
./webodm.sh restart
```

On some platforms (eg. Windows), if you want to manage plugins, you will need to make sure that the `./plugins` directory can be mounted as a docker volume and then pass the `--mount-plugins-volume` flag to `webodm.sh`. Check the docker documentation.

## Customizing and Extending

Small customizations such as changing the application colors, name, logo, or addying custom CSS/HTML/Javascript can be performed directly from the Customize -- Brand/Theme panels within WebODM. No need to fork or change the code.

More advanced customizations can be achieved by writing [plugins](https://github.com/OpenDroneMap/WebODM/tree/master/plugins). This is the preferred way to add new functionality to WebODM since it requires less effort than maintaining a separate fork. The plugin system features server-side [signals](https://github.com/OpenDroneMap/WebODM/blob/master/app/plugins/signals.py) that can be used to be notified of various events, a ES6/React build system, a dynamic [client-side API](https://github.com/OpenDroneMap/WebODM/tree/master/app/static/app/js/classes/plugins) for adding elements to the UI, a built-in data store, an async task runner, a GRASS engine, hooks to add menu items and functions to rapidly inject CSS, Javascript and Django views.

The plugin system is still in beta. The best source of documentation currently is to look at existing [code](https://github.com/OpenDroneMap/WebODM/tree/master/plugins). If a particular hook / entrypoint for your plugin does not yet exist, [request it](https://github.com/OpenDroneMap/WebODM/issues). We are adding hooks and entrypoints as we go.

To create a plugin simply copy the `plugins/test` plugin into a new directory (for example, `plugins/myplugin`), then modify `manifest.json`, `plugin.py` and issue a `./webodm.sh restart`.

## API Docs

See the [API documentation page](http://docs.webodm.org).

## ODM, NodeODM, WebODM... what?

The [OpenDroneMap project](https://github.com/OpenDroneMap/) is composed of several components.

- [ODM](https://github.com/OpenDroneMap/ODM) is a command line toolkit that processes aerial images. Users comfortable with the command line are probably OK using this component alone.
- [NodeODM](https://github.com/OpenDroneMap/NodeODM) is a lightweight interface and API (Application Program Interface) built directly on top of [ODM](https://github.com/OpenDroneMap/ODM). Users not comfortable with the command line can use this interface to process aerial images and developers can use the API to build applications. Features such as user authentication, map displays, etc. are not provided.
- [WebODM](https://github.com/OpenDroneMap/WebODM) adds more features such as user authentication, map displays, 3D displays, a higher level API and the ability to orchestrate multiple processing nodes (run jobs in parallel). Processing nodes are simply servers running [NodeODM](https://github.com/OpenDroneMap/NodeODM).

![webodm](https://cloud.githubusercontent.com/assets/1951843/25567386/5aeec7aa-2dba-11e7-9169-aca97b70db79.png)

In general, follow these guidelines to find out what you should use:

I am a... | Best choice
--------- | -----------
End user, I'm not really comfortable with the command line | [WebODM](https://github.com/OpenDroneMap/WebODM)
End user, I like shell commands, I need to process images for myself. I use other software to display processing results |  [ODM](https://github.com/OpenDroneMap/ODM)
End user, I can work with the command line, but I'd rather not. I use other software to display processing results  |  [NodeODM](https://github.com/OpenDroneMap/NodeODM)
End user, I need a drone mapping application for my organization that everyone can use. | [WebODM](https://github.com/OpenDroneMap/WebODM)
Developer, I'm looking to build an app that displays map results and takes care of things like permissions | [WebODM](https://github.com/OpenDroneMap/WebODM)
Developer, I'm looking to build an app that will stay behind a firewall and just needs raw results | [NodeODM](https://github.com/OpenDroneMap/NodeODM)

## Roadmap
- [X] User Registration / Authentication
- [X] UI mockup
- [X] Task Processing
- [X] 2D Map Display 
- [X] 3D Model Display
- [ ] NDVI display
- [X] Volumetric Measurements
- [X] Cluster management and setup.
- [ ] Mission Planner
- [X] Plugins/Webhooks System
- [X] API
- [X] Documentation
- [ ] Android Mobile App
- [ ] iOS Mobile App
- [ ] Processing Nodes Volunteer Network
- [X] Unit Testing
- [X] SSL Support

Don't see a feature that you want? [Help us make it happen](/CONTRIBUTING.md). 

## Getting Help

We have several channels of communication for people to ask questions and to get involved with the community:

 - [OpenDroneMap Community Forum](http://community.opendronemap.org/c/webodm)
 - [Report Issues](https://github.com/OpenDroneMap/WebODM/issues)

We also have a [Gitter Chat](https://gitter.im/OpenDroneMap/web-development), but the preferred way to communicate is via the [OpenDroneMap Community Forum](http://community.opendronemap.org/c/webodm).

## Support the Project

There are many ways to contribute back to the project:

 - Help us test new and existing features and report [bugs](https://www.github.com/OpenDroneMap/WebODM/issues) and [feedback](http://community.opendronemap.org/c/webodm).
 - [Share](http://community.opendronemap.org/c/datasets) your aerial datasets.
 - Help answer questions on the community [forum](http://community.opendronemap.org/c/webodm) and [chat](https://gitter.im/OpenDroneMap/web-development).
 - ⭐️ us on GitHub.
 - Spread the word about WebODM and OpenDroneMap on social media.
 - While we don't accept donations, you can purchase an [installer](https://webodm.org/download#installer) or a [premium support package](https://webodm.org/services#premium-support).
 - Become a contributor (see below to get free swag 🤘)

## Become a Contributor

The easiest way to get started is to take a look at our list of [outstanding issues](https://github.com/OpenDroneMap/WebODM/labels/help%20wanted) and pick one. You can also fix/improve something entirely new based on your experience with WebODM. All ideas are considered and people of all skill levels are welcome to contribute. 

You don't necessarily need to be a developer to become a contributor. We can use your help to write better documentation and improve the user interface texts and visuals. 

If you know how to code, we primarily use Python (Django), Javascript (React), HTML and SCSS. See the [Development Quickstart](http://docs.webodm.org/#development-quickstart) and [Contributing](/CONTRIBUTING.md) documents for more information.

To make a contribution, you will need to open a pull request ([here's how](https://github.com/Roshanjossey/first-contributions#fork-this-repository)). To make changes to WebODM, make a clone of the repository and run `./devenv.sh start`.

If you have questions visit us on the [forum](http://community.opendronemap.org/c/webodm) and we'll be happy to help you out with your first contribution.

When your first pull request is accepted, don't forget to fill [this form](https://goo.gl/forms/PZkiPPeNKUHNz0qe2) to get your **free** WebODM T-Shirt 🤘

<img src="https://user-images.githubusercontent.com/1951843/36511023-344f86b2-1733-11e8-8cae-236645db407b.png" alt="T-Shirt" width="50%">

## Architecture Overview

WebODM is built with scalability and performance in mind. While the default setup places all databases and applications on the same machine, users can separate its components for increased performance (ex. place a Celery worker on a separate machine for running background tasks).

![Architecture](https://user-images.githubusercontent.com/1951843/36916884-3a269a7a-1e23-11e8-997a-a57cd6ca7950.png)

A few things to note:
 * We use Celery workers to do background tasks such as resizing images and processing task results, but we use an ad-hoc scheduling mechanism to communicate with NodeODM (which processes the orthophotos, 3D models, etc.). The choice to use two separate systems for task scheduling is due to the flexibility that an ad-hoc mechanism gives us for certain operations (capture task output, persistent data and ability to restart tasks mid-way, communication via REST calls, etc.).
 * If loaded on multiple machines, Celery workers should all share their `app/media` directory with the Django application (via network shares). You can manage workers via `./worker.sh`


## Run the docker version as a Linux Service

If you wish to run the docker version with auto start/monitoring/stop, etc, as a systemd style Linux Service, a systemd unit file is included in the service folder of the repo.

This should work on any Linux OS capable of running WebODM, and using a SystemD based service daemon (such as Ubuntu 16.04 server for example).

This has only been tested on Ubuntu 16.04 server.

The following pre-requisites are required:
 * Requires odm user
 * Requires docker installed via system (ubuntu: `sudo apt-get install docker.io`)
 * Requires screen to be installed
 * Requires odm user member of docker group
 * Required WebODM directory checked out to /webodm
 * Requires that /webodm is recursively owned by odm:odm
 * Requires that a Python 3 environment is used at /webodm/python3-venv

If all pre-requisites have been met, and repository is checked out to /opt/WebODM folder, then you can use the following steps to enable and manage the service:

First, to install the service, and enable the services to run at startup from now on:
```bash
sudo systemctl enable /webodm/service/webodm-gunicorn.service
sudo systemctl enable /webodm/service/webodm-nginx.service
```

To manually start/stop the service:
```bash
sudo systemctl stop webodm-gunicorn
sudo systemctl start webodm-gunicorn
```

To manually check service status:
```bash
sudo systemctl status webodm-gunicorn
```

## Run it natively

WebODM can run natively on Windows, MacOS and Linux.

Ubuntu 16.04 LTS users can refer to [this community script](/contrib/ubuntu_1604_install.sh) to install WebODM natively on a new machine.

A commercial installer with technical support [is also available](https://webodm.org/server).

To run WebODM, you will need to install:
 * PostgreSQL (>= 9.5)
 * PostGIS 2.3
 * Python 3.5
 * GDAL (>= 2.1)
 * Node.js (>= 6.0)
 * Nginx (Linux/MacOS) - OR - Apache + mod_wsgi (Windows)
 * Redis (>= 2.6)

On Linux, make sure you have:

```bash
apt-get install binutils libproj-dev gdal-bin nginx
```

On Windows use the [OSGeo4W](https://trac.osgeo.org/osgeo4w/) installer to install GDAL. MacOS users can use:

```
brew install postgres postgis
```

Then these steps should be sufficient to get you up and running:

```bash
git clone --depth 1 https://github.com/OpenDroneMap/WebODM
```

Create a `WebODM/webodm/local_settings.py` file containing your database settings:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.contrib.gis.db.backends.postgis',
        'NAME': 'webodm_dev',
        'USER': 'postgres',
        'PASSWORD': 'postgres',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

From psql or [pgadmin](https://www.pgadmin.org), connect to PostgreSQL, create a new database (name it `webodm_dev`), connect to it and set the [postgis.enable_outdb_rasters](http://postgis.net/docs/manual-2.2/postgis_enable_outdb_rasters.html) and [postgis.gdal_enabled_drivers](http://postgis.net/docs/postgis_gdal_enabled_drivers.html) settings:

```sql
ALTER SYSTEM SET postgis.enable_outdb_rasters TO True;
ALTER SYSTEM SET postgis.gdal_enabled_drivers TO 'GTiff';
```

Start the redis broker:

```bash
redis-server
```

Then:

```bash
pip install -r requirements.txt
sudo npm install -g webpack
sudo npm install -g webpack-cli
npm install
webpack --mode production
python manage.py collectstatic --noinput
chmod +x start.sh && ./start.sh --no-gunicorn
```

Finally, start at least one celery worker:

```bash
./worker.sh start
```

The `start.sh` script will use Django's built-in server if you pass the `--no-gunicorn` parameter. This is good for testing, but bad for production. 

In production, if you have nginx installed, modify the configuration file in `nginx/nginx.conf` to match your system's configuration and just run `start.sh` without parameters. 

Windows users should refer to [this guide](https://docs.djangoproject.com/en/1.11/howto/deployment/wsgi/modwsgi/) to install Apache + mod_wsgi and run gunicorn:

```bash
gunicorn webodm.wsgi --bind 0.0.0.0:8000 --preload
```

If you are getting a `rt_raster_gdal_warp: Could not create GDAL transformation object for output dataset creation`, make sure that your PostGIS installation has PROJ support:

```sql
SELECT PostGIS_Full_Version();
```

You may also need to set the environment variable PROJSO to the .so or .dll projection library your PostGIS is using. This just needs to have the name of the file. So for example on Windows, you would in Control Panel -> System -> Environment Variables add a system variable called PROJSO and set it to libproj.dll (if you are using proj 4.6.1). You'll have to restart your PostgreSQL service/daemon after this change. [http://postgis.net/docs/manual-2.0/RT_ST_Transform.html](http://postgis.net/docs/manual-2.0/RT_ST_Transform.html)

If you are using Windows and are unable to go past the `pip install -r requirements.txt` command because of an error regarding zlib and Pillow, manually edit the `requirements.txt` file, remove the Pillow requirement and run:

```bash
easy_install pillow
pip install -r requirements.txt
```

On Windows make sure that all of your PATH environment variables are set properly. These commands:

```bash
python --version
pip --version
npm --version
gdalinfo --version
redis-server --version
```
Should all work without errors.

## Run it on the cloud (Google Compute, Amazon AWS)

12 steps, to have WebODM running on a cloud instance.

These steps are for Google Cloud, but can also be used for Amazon AWS, and other cloud platforms with small modifications:

1. Launch a Google Cloud instance of Ubuntu 18.0 LTS.
2. Open the SSH terminal - Google offers SSH via the website.
3. Run sudo apt-get update
4. Run sudo apt-get upgrade
5. Run sudo apt-get install docker-compose
6. Run sudo apt-get install python-pip
7. Run git clone https://github.com/OpenDroneMap/WebODM --config core.autocrlf=input --depth 1
8. cd WebODM (Linux is case sensitive)
9. sudo ./webodm.sh start
10. You now can access webodm via the public IP address for your google instance. Remember the default port of 8000.
11. Check that your instance's firewall is allowing inbound TCP connections on port 8000! If you forget this step you will not be able to connect to WebODM.
12. Open http://GooglepublicIPaddressforyourinstance:8000

To setup the firewall on Google Cloud, open the instance, on the middle of the instance settings page find NIC0. Open it, and then add the TCP Port 8000 for ingress, and egress on the Firewall.


