# Deployment guide

FutureGateway can be installed on top of the following environments:
* Enterprise Linux 6/7
* Debian (Ubuntu)
* MacOSX
The installation procedure is managed by a set of bash scripts operating at different levels. Low level scripts are in charge to install FutureGateway components targeting the different operating system. High level scripts manage low level scripts to install the system for a specific operating system.
Installation process may differ in case there exists a specific OS/Architecture hi-level isntallation script.

## Using low level scripts
Architectures not targeted by hi-level scripts maybe installed as well, directly using low level scripts. These installation files can be used on all those OS platforms using as package management tools: yum, apt and brew; thus: EL OSes (i.e RedHat, CentOS, Debian, MacOSX and potentially Windows under Cygwin environment).
In order to use low level scritps, the user has to download from the git repository all files having the name 'setup\_\<component\>.sh'. Then the user can install the system just executing each script in the order:

1. ./setup_FGPortal.sh - Install the core components of the system and preparing the necessary environment
2. ./setup_JSAGA.sh - Provides a complete JSAGA installation with binaries and libraries
3. ./setup_GridEngine.sh - Install the Grid&Cloud Engine (necessary to target SSH, EMI/gLite, rOCCI infrastructures) 
4. ./setup_OCCI.sh - Install the OCCI CLI with GSI necessary packages
5. ./setup_FGService.sh - Execute only in case you want to run FutureGateway as a service

Before execute any 'setup\_\<component\>.sh' script; the user has to configure the file setup\_config.sh file to override default configuration settings written at the top of each setup file.
Once installation files have been executed; it is necessary to log-out and login again; so that the new environment will be ready and the two APIServer components (fgAPIServer and APIServerDaemon) can be installed from sources with the following steps:

1. Download fgAPIServer files from gitHub in the directory: $FGLOCATION/fgAPIServer
2. Execute fgapiserver.py script inside a dedicated screen section or confiure wsgi to run it
3. Download APIServerDaemon files from gitHub in the directory: $FGLOCATION/APIServerDaemon
4. Go inside the APIServerDaemon directory and compile java code with: ant all
5. Copy the generated war file into $CATALINA_HOME/webapps directory and check on $CATALINA_HOME/logs/catalina.out file that the installation is successful

At this stage the user has a complete and operating FutureGateway environment.
As reference the reader can check existing hi-level scripts, in particular the file fgSetup.sh

## Ubuntu LTS 14.04 Server 

In order to install the FutureGateway, just execute as root user:

```sh
# IP=$(ifconfig | grep  -A 2 eth0 | grep inet\ addr | awk -F':' '{ print $2 }' | awk '{ print $1 }' | xargs echo)
# echo "$IP    futuregateway" >> /etc/hosts
# adduser --disabled-password --gecos "" futuregateway
# mkdir -p /home/futuregateway/.ssh
# chown futuregateway:futuregateway /home/futuregateway/.ssh
# wget https://github.com/FutureGateway/PortalSetup/raw/master/Ubuntu_14.04/fgSetup.sh
# chmod +x fgSetup.sh
# cat /dev/zero | ssh-keygen -q -N ""
# cat /root/.ssh/id_rsa.pub >> /home/futuregateway/.ssh/authorized_keys
# echo "#FGSetup remove the following after installation" >> /etc/sudoers
# echo "ALL  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
# ./fgSetup.sh futuregateway futuregateway <your ssh port> $(cat /root/.ssh/id_rsa.pub)
```
Before to execute `fgSetup.sh` it is recommended to open it and verify inside its content, the default settings. Executing the last statement above, the installation procedure should start and it requires a while to complete.

## FedCloud Installation

The FutureGateway portal can be instantiated by an existing EGI FedCloud virtual appliance named: [FutureGateway][FGAPPDB].
The virtual appliance is based on an Ubuntu-server 14.04 and it requires a specific cloud-init' user-data file in order to setup it properly while instantiating the machine.
The principal aim of the FutureGateway virtual appliance is to allow Science Gateway application developers to make practice with FutureGateway REST APIs without taking care of the whole system intallation.

Below the commands to correctly instantiate the VM:

`OCCI_RES=$(occi -e $OCCI_ENDPOINT --auth x509 --user-cred $USER_CRED --voms $VOMS --action create --resource compute --mixin os_tpl#$OS_TPL --mixin resource_tpl#$RESOURCE_TPL --attribute occi.core.title="fgocci2" --context user_data="file://$HOME/userdata.txt"); echo "Resourcce: $OCCI_RES"`

The contextualization file `userdata.txt` has to be customized before to execute the `occi` command line. In particular the user must provide its own ssh public key under cloud-init key `ssh-authorized-keys`. Another point to customize is inside the `write_files` directive for the file `/root/installFG.sh`. The following sed commands must be configured:

`sed s/TOMCATUSR=\"tomcat\"/TOMCATUSR=\"<tomcat_admin_user>\"/`
`sed s/TOMCATPAS=\"tomcat\"/TOMCATUSR=\"<tomcat_admin_password>\"/`

replace text inside the `<...>` brackets with your preferred Tomcat service admin username and password.
Further and more sofisticated customizations could be done in the same fashion mofifying the downloaded script `fgSetup.sh`

In case it is needed to assign a public IP to the given resource, execute:

`occi --endpoint $OCCI_ENDPOINT --auth x509 --user-cred $USER_CRED --voms --action link --resource $OCCI_RES --link /network/public`

### Security considerations
Although the VM has been configured to limit hackers exposure, it is warmly suggested to comply with the EGI FedCloud [directives][EGIFCDR]

[FGAPIFE]: <https://github.com/FutureGateway/fgAPIServer>
[FGASRVD]: <https://github.com/FutureGateway/APIServerDaemon>
[EGIFCDR]: <https://wiki.egi.eu/wiki/Virtual_Machine_Image_Endorsement#Hardening_guidelines>

# Suggested procedures
The installation scritps will instantiate the full FutureGateway environment extracting anything from GITHub, so that fresh installations will contain the latest available packages version. To know about the status or the end of the installation procedure, please check the output of the scripit.
Once finished the installation it is important to exit from any ssh connection active before the installation procedure and re-log again. During the re-connection, ssh will recognize a host identification change, then proceed to accept the new identity. In case the script have been executed from root it is enough to change the user with `su - futuregateway`.

In order to test FutureGateway REST APIs, several services should be started before; in particular:

1. The REST APIs [front-end][FGAPPDB]
2. The API Server Daemon [ServerDaemon][FGASRVD]

## REST APIs front-end (fgAPIServer)
In a production environment the API server front-end must be configured with a dedicated wsgi configuration inside the web server. However for testing purposes the front-end can be executed in stand-alone mode with the following set of commands:

* Instantiate a screen section:
`screen -S fgAPIServer`
* Execute the API REST front-end:
`cd $FGLOCATION/fgAPIServer`
`cd $FGLOCATION/fgAPIServer`
`./fgapiserver.py`
Detach with \<ctrl-a\>\<ctrl-d\>
Reattach the front-end process anytime with `screen -r fgAPIServer`

An example of wsgi configuration in site configuration as reported below:
```
<IfModule wsgi_module>
  WSGIDaemonProcess fgAPIServer  user=futuregateway group=futuregateway  processes=5 threads=10 home=/home/futuregateway
  WSGIProcessGroup futuregateway
  WSGIScriptAlias /apis /home/futuregateway/FutureGateway/fgAPIServer/fgapiserver.wsgi
  WSGIPassAuthorization On

  <Directory /home/futuregateway/FutureGateway/fgAPIServer>
    WSGIProcessGroup fgAPIServer
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

* Startup application server:
`start_tomcat`. To manage daemon activity you can use the Tomcat manager front-end with `http://<VM_IP>:8080/manager` (default credentials are tomcat/tomcat).To stop Tomcat you can use `stop_tomcat` then please verify its java process with `ps -ef | grep tomcat | grep java` if the process still perist you may use '`killjava` command.

* Monitor the APIServer daemon app server activity:
`tail -f $CATALINA_HOME/logs/catalina.out`
It is important during development phases to constatly monitor the APIServer daemon activity, to accomplish that it is enough to have a look inside the application server log file.

* Monitor the APIServer daemon activity:
`tail -f $CATALINA_HOME/webapps/APIServerDaemon/WEB-INF/logs/APIServerDaemon.log`

* Monitor the GridEngine activity:
`tail -f $CATALINA_HOME/webapps/APIServerDaemon/WEB-INF/logs/GridEngineLog.log

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




