# Deployment guide

FutureGateway can be installed on top of the following environments:
* Enterprise Linux 6/7
* Debian (Ubuntu)
* MacOSX

Thre are currently two possible ways to install the FutureGateway, one based on setup scripts, the other way using Docker containers. Both installation procedures will setup the three FutureGateway components:

* The database
* The APIServer front-end (fgAPIServer)
* The APIServer daemon (APIServerDaemon) and its adaptors

Beside the core components, GUI oriented services may be available such as

* Liferay7

GUI components may be not aligned with the core components or could be not fully working. It is strongly suggested to check their contents before to install them.


# Script based installation procedure

The script based installation procedure currently supports the MacOS X and Ubuntu (deb) OSes. The installation script structure has been designed to easily integrate other platforms.
Another importan feature of the installation scripts is related to the possibility of installing each FG component in a dedicated host.

## Using scripts

The use of the script installation is easy and made of two steps. First configure the services modifying the file: `setup_config.sh`. Then executing the script `setup_futuregateway.sh`

The `setup_config.sh` contains all the configurable environments variables related to the FutureGateway components:

* The database (fgdb)
* The APIServer front-end (fgAPIServer)
* The APIServer daemon (APIServerDaemon) and its adaptors
* Liferay (optional)

The main setup procedure script `setup_futuregateway.sh` calls the proper isntallation script relying on the kind of package manager found on the target host. At the moment the foreseen package managers are:

* brew - for MacOS X platforms
* apt - For Debian based platforms (Ubuntu 14.04 LTS)
* yum - For RedHat based platforms (CentOS 7) (not yet available)

The installation process foresees the following steps:

1. Identify the necessary components to install and the whole FutureGateway services topology. In case more hosts are involved in the setup process, please ensure to run the setup from a host able to connect via ssh ach node passwordlessly, properly exchanging SSH keys. Early phases of the setup will try to identify any missing mandatory configuration.
2. Modify the setup\_config.sh file configuring each FutureGateway service as designed in the previous step. Each FutureGateway component contains its own specific settings inside the setup\_config.sh script. Any FG user specified in the configurion file setup\_config.sh must be already present in its host system with passwordless sudo authorization as well as SSH key exchange with the installation node.
3. From the installation host, execute the script setup\_futuregateway.sh. The first time the setup procedure will install from Git all selected components, while further executions will try to upgrade the components and update its configurations accordingly with the values placed in the file setup\_config.sh

### Instasllation test (virtualization)
Under the [fgSetup/test][FGSETUP_TEST] installation folder, it is possible to test the installation using a Docker container.
To perform the installation test, just execute:

```
make image
make run
```

This process may take a while, because both image preparation and the installation of the FG components may requires a while, especially for the APIServerDaemon. Once the installation is accomplished, to execute the tests, execute inside the containerized FG the script:

```
./do_tests.sh
```

### Suggested procedures
The installation scritps will instantiate the full FutureGateway environment extracting anything from GITHub, so that fresh installations will contain the latest available packages version or in alternative the version specified in the setup scripts. To know about the status or the end of the installation procedure, please check the output of the scripit.
Once finished the installation it is important to exit from any ssh connection active before the installation procedure and re-log again. During the re-connection, ssh will recognize a host identification change, then proceed to accept the new identity. In case the script have been executed from root it is enough to change the user with `su - futuregateway`.

# Containers
Docker Containers are totally matching the FutureGateway philosophy to provide the most possible flexible and customisable solution for the final user, for this reason for each FG component a dedicated docker container has been developed.
There are two possibilities to install FG using Docker containers.

1. The first uses the docker-compose and containers officially deployed in the [Docker Hub][FGDKRHUB]
2. The classic docker container installation using Dockerfiles

## Docker compose
In order to install you own FG in a containerised environment able to support docker-compose, the following commands are enough to install and also test the core APIs.

```bash
# 1) Retrieve FG setup scripts
git clone https://github.com/FutureGatewayFramework/fgSetup.git

# 2) Move inside the extracted folder up the ‘docker’ dir:
cd fgSetup/docker

# 3) Prepare the docker compose file with:
./setup_futuregateway.sh

# 4) Startup the docker-compose
docker-compose up -d
```

At this stage the FutureGateway is almost ready.

```bash
# 5) In order to test the installed FG instance, just execute:
./exec_futuregateway_test.sh
```

If everything will be fine you will see the FG operating its core APIs successfully

### Notes
When the docker-compose starts, the DB instance may take a while in order to be ready. It may happen that other modules (fgapiserver, apiserverdaemon) will fail attempting to connect the DB when they start. This situation is handled by the `/exec_futuregateway_test.sh` by the function `align_installation`, which simply restart the relative docker instances.
The compose instantiates the ‘sshnode’ container that is used to perform the test job submission; this node may be extended by the compose including a volume and/or instantiating more instances.
The volume could be useful to preserve data persistency if necessary, while the multiple instances may be useful to scale horizontally the job submission.
The `docker-compose.yml` file can be used by docker swarm in order to guarantee the scalability. Tests on swarm haven’t been done yet.

## Docker containers
The Docker based installation foresees a different container for each FutureGateway component and all necessary files to setup the system are available on GitHub under [fgSetup/docker][FGSETUP_DOCKER] directory.
The following chapters are reporting the correct procedure to install each FG component and they have to be executed in the same order as they appear in the instructions.
The common procedure for all FG components is to extract the fgSeutp repository preferably cloning it from its [master branch][FGSETUP_GIT], configure the Dockefile and execute several Makefile receipts. For this reason GNU make is required at the moment to install FG using Docker Containers. Makefile themselves may require to be configured on top in accordance with configuration setting wrote in the Dockerfiles.

## fgdb
Execute the following commands to setup the FutureGateway database.
Before to execute the `make image` step, please ensure that environment variables inside the `Dockerfile` are matching your environment.
The use of default values should be enough to ensure a stable installation.

```sh
cd $HOME/FutureGatewayFramework/fgSetup/docker/fgdb
sudo make image
sudo make run
```

## fgAPIServer
As for the database components the fgAPIServer can be installed with the following steps:

```sh
cd $HOME/FutureGatewayFramework/fgSetup/docker/fgapiserver
sudo make image
sudo make run
```

The latter `make run` command can be replaced with `make testrun` in the case it is required to apply changes to the python code providing the REST APIs. Using this Makefile receipt, it is important to check environment variables specified at the top of the Makefile itself. These variables will be used to create on the fly the `fgapiserver.conf` file.

## APIServerDaemon
As for the previous component the APIServerDaemon component can be installed with the following steps:

```sh
cd $HOME/FutureGatewayFramework/fgSetup/docker/apiserverdaemon
sudo make image
sudo make run
```

## SSH node
This is a node that could be installed to perform a basic tests on FG exploiting the GridEngine ExecutorInterface using the JSAGA ssh adaptor.
The following commans create and configure this node with the APIServerDaemon.

```sh
cd $HOME/FutureGatewayFramework/fgSetup/docker/sshnode
sudo make image
sudo make run
sudo make apiserverdaemon
```
The latter Makefile receipt allows SSH connections from APIServerDaemon using login/password to the SSH node.

## Liferay7
Currently under development/testing

##	fghttpd
This is not properly a FG component at the moment, but it could be an important container to properly implement a production quality FG system. The fghttpd provides an Apache web server where configure the ReverseProxy (Gateway Server).
This solution ensures security and scalability.

```sh
cd $HOME/FutureGatewayFramework/fgSetup/docker/fghttpd
sudo make image
sudo make run
```

### Notes
It is possible to override standard settings by using environment variables or directly modifying configuration settings stored in the database once the service starts. This feature is not available for the APIServerDaemon component.

## Using ansible playbooks

A third installation has been supported in the past foreseegin the use of ansible playbooks, however this has been deprecated. There exists a third party [repository][EOSCFG] that is still supporting them.

# Testing components
In order to test FutureGateway REST APIs, several services should be started before; in particular:

1. The REST APIs [front-end][FGAPISRV]
2. The API Server Daemon [ServerDaemon][APISRVDMN]

## REST APIs front-end (fgAPIServer)
In a production environment the API server front-end must be configured with a dedicated wsgi configuration inside the web server. However for testing purposes the front-end can be executed in stand-alone mode with the following set of commands:

* If not yet installed by a setup script, extract from git the sources:
`git clone https://github.com/indigo-dc/fgAPIServer $FGLOCATION/fgAPIServer`
* Instantiate a screen section:
`screen -S fgAPIServer`
* Execute the API REST front-end:
`cd $FGLOCATION/fgAPIServer`
`cd $FGLOCATION/fgAPIServer`
`./fgapiserver.py`
Detach with `<ctrl-a><ctrl-d>`
Reattach the front-end process anytime with `screen -r fgAPIServer`

An example of wsgi configuration in site configuration as reported below:
```
<IfModule wsgi_module>
			WSGIDaemonProcess fgapiserver  user=futuregateway group=futuregateway  processes=5 threads=10 home=/home/futuregateway/fgAPIServer
			WSGIProcessGroup futuregateway
			WSGIScriptAlias /apis /home/futuregateway/FutureGateway/fgapiserver/fgapiserver.wsgi

			<Directory /home/futuregateway/FutureGateway/fgapiserver>
			  WSGIProcessGroup fgapiserver
			  WSGIApplicationGroup %{GLOBAL}
			  Order deny,allow
			  Allow from all
			  Options All
			  AllowOverride All
			  Require all granted
			</Directory>
		</IfModule>
```

An example of wsgi configuration is also available under `$FGLOCATION/fgAPIServer.conf`.
Enabling the front-end to work with wsgi, it is no more necessary to use the screen section. To switch off the screen execution, just turn off the `ENABLEFRONTEND` (place zero value) flag in the service script file `/etc/init.d/futuregateway`.

### APIServer configuration

Most of APIServer configurations are included inside the `fgapiserver.conf` configuration file. Before to execute the front-end it is important to setup this file properly.
The configuration file consists of two kind of settings: the ones related to the APIServer and the ones related to the database. Below a description of available settings:

APIServer settings:

* `fgapiver`: API Specifications version (default v1.0)
* `fgapiserver_name`: Name of the front-end service (just informative setting)
* `fgapisrv_host`: Use 0.0.0.0 to open the service to all hosts
* `fgapisrv_port`: Port number where the API server will be listening
* `fgapisrv_debug`: Enable/Disable debugging mode
* `fgapisrv_iosandbox`: Specify the mount point of task IO/Sandboxes
* `fgapisrv_geappid`: Grid&Cloud Engine application id
* `fgjson_indent`: JSON indent number of spaces
* `fgapisrv_key`: HTTPS mode; service certificate key path
* `fgapisrv_crt`: HTTPS mode; service certificate path
* `fgapisrv_logcfg`: Path to the log configuration file (use full path in wsgi conf)
* `fgapisrv_dbver`: APIServer database schema version
* `fgapisrv_secret`: Key value for baseline authN/Z service
* `fgapisrv_notoken`: Avoid token check
* `fgapisrv_notokenusr`:  APIServer user to be used when operating with no tokens
* `fgapisrv_lnkptvflag`:  Portal Token Vallidation service switch
* `fgapisrv_ptvendpoint`: PTV token check endpoint 
* `fgapisrv_ptvuser`: PTV Basic authentication user name
* `fgapisrv_ptvpass`: PTV Basic authentication password
* `fgapisrv_ptvdefusr`: PTV default user when token does not map with any registered APIServer user
* fgapisrv_ptvmapfile`: JSON file containing user mapping configuration 
`
Database settings:

* `fgapisrv_db_host`: Database hostname
* `fgapisrv_db_port`: Database port number
* `vfgapisrv_db_user`: Database user
* `fgapisrv_db_pass`: Database password
* `fgapisrv_db_name`: Database name

## APIServer Daemon (APIServerDaemon)
The API Server Daemon consists of a web application, so that it is necessary to startup the application server (Tomcat). The virtual appliance is already configured to install and execute the daemon during the application server startup.
To startup the application server you may use the standard scripts provided with Tomcat or you may use the 'start\_tomcat' utility:

* If not yet installed by a setup script, extract from git the sources:
`git clone https://github.com/indigo-dc/APIServerDaemon $FGLOCATION/APIServerDaemon`
* If necessary compile its code going inside `$FGLOCATION/APIServerDaemon` direcory and executing `ant all`. Then copy the generated war file in the directory `$CATALINA_HOME/webapps/`
* Startup application server:
`start_tomcat`. To manage daemon activity you can use the Tomcat manager front-end with `http://<VM_IP>:8080/manager` (default credentials are tomcat/tomcat).To stop Tomcat you can use `stop_tomcat` then please verify its java process with `ps -ef | grep tomcat | grep java` if the process still perist you may use '`killjava` command.

* Monitor the APIServer daemon app server activity:
`tail -f $CATALINA_HOME/logs/catalina.out`
It is important during development phases to constatly monitor the APIServer daemon activity, to accomplish that it is enough to have a look inside the application server log file.

* Monitor the APIServer daemon activity:
`tail -f $CATALINA_HOME/webapps/APIServerDaemon/WEB-INF/logs/APIServerDaemon.log`

* Monitor the GridEngine activity:
`tail -f $CATALINA_HOME/webapps/APIServerDaemon/WEB-INF/logs/GridEngineLog.log`

### APIServer Daemon configuration
Also the APIServer daemon comes with several configuration settings stored in file: `$CATALINA_HOME/webapps/APIServerDaemon/WEB-INF/classes/it/infn/ct/APIServerDaemon.properties` and below reported:

* `apisrv_dbhost`: APIServer database host
* `apisrv_dbport`: APIServer database port number
* `apisrv_dbuser`: APIServer database user name
* `apisrv_dbpass`: APIServer database user password
* `apisrv_dbname`: APIServer database name
* `apisrv_dbver`: APIServer database schema version
* `asdMaxThreads`:  Maximum number of allowed threads
* `asdCloseTimeout`: Timeout during daemon stopping to kill pending processes
* `gePollingDelay`: How many seconds the polling thread waits before starting a new cycle
* `gePollingMaxCommands`: Maximum number of queue commands to extract from the queue for each polling cycle
* `asControllerDelay`: How many seconds the controller waits before starting a new cycle
* `asControllerMaxCommands`: How many commands should be extracted from queue per cycle
* `asTaskMaxRetries`: Maximum number of queue commands retries in case of failure
* `asTaskMaxWait`: How many seconds a failed task has to wait after its execution for the next retry
* `utdb_jndi`: Grid and Cloud Engine database connection pool name (goes alternatively to below utdb_* settings)
* `utdb_host`: Grid and Cloud Engine database database host
* `utdb_port`: Grid and Cloud Engine database database port number
* `utdb_user`: Grid and Cloud Engine database database user name
* `utdb_pass`: Grid and Cloud Engine database database user password
* `utdb_name`: Grid and Cloud Engine database database name


## Executor Interfaces configuration

Executor interfaces may require to specify several settings

### ToscaIDC

This executor interface needs to configure several settings related to the PTV service.
The values have to be specified inside the `web/WEB-INF/classes/it/infn/ct/ToscaIDC.properties` file in particular the following values have to configured:

* `fgapisrv_ptvendpoint`: PTV hostname/address
* `fgapisrv_ptvuser`: PTV username
* `fgapisrv_ptvuser`: PTV password

* `fgapisrv_frontend`: API server front-end entry point
* `fgapisrv_ptvtokensrv`: PTV service name to get a token
* `fgapisrv_ptvuser`: PTV basic authentication username
* `fgapisrv_ptvpass`: PTV basic authentication password


[FGAPISRV]: <https://github.com/FutureGatewayFramework/fgAPIServer>
[FGAPISRV_GIT]: <https://github.com/FutureGatewayFramework/fgAPIServer.git>
[APISRVDMN]: <https://github.com/FutureGatewayFramework/APIServerDaemon>
[APISRVDMN_GIT]: <https://github.com/FutureGatewayFramework/APIServerDaemon.git>
[FGAPISRV_GIT]: <https://github.com/FutureGatewayFramework/fgAPIServer>
[FGSETUP]: <https://github.com/FutureGatewayFramework/fgSetup>
[FGSETUP_GIT]: <https://github.com/FutureGatewayFramework/fgSetup.git>
[FGSETUP_DOCKER]: <https://github.com/FutureGatewayFramework/fgSetup/docker>
[FGSETUP_TEST]: <https://github.com/FutureGatewayFramework/fgSetup/test>
[EOSCFG]: <https://github.com/tzok/eosc-futuregateway>
[FGDKRHUB]: <https://hub.docker.com/u/futuregateway>
