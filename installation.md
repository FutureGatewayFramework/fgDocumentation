# Deployment guide

FutureGateway can be installed on top of the following environments:
* Enterprise Linux 6/7
* Debian (Ubuntu)
* MacOSX

Thre are currently two possible ways to install the FutureGateway, one is related to the older installation procedure totally made of shell scripts and a new DevOps based procedure based on Ansible playbooks.

# New installation procedure

The new installation procedure maintains as well the script based procedures, but it also provides a DevOps approach offering several Ansible playbooks, each for a different FutureGateway component:

* The database
* The APIServer front-end (fgAPIServer)
* The APIServer daemon (APIServerDaemon) and its adaptors
* Liferay (Not yet available)

The script approach currently supports the MacOS X only and it has beeen re-engineered to offer a more elastic structure allowing for instance to separate FutureGateway components into different hosts. The script way has been also maintained to cope all those cases where the FutureGateway has to run for unsupported platforms such as MacOS X.

The new installation procedure comes inside the repository named *fgSetup*

## Using scripts

The use of the script installation is easy and made of two steps. First configure the services modifying first the file: `setup_config.sh`
Then executing the script `setup_futuregateway.sh`

The `setup_config.sh` contains all the configurable environments related to the FutureGateway components:

* The database
* The APIServer front-end (fgAPIServer)
* The APIServer daemon (APIServerDaemon) and its adaptors
* Liferay

The main setup procedure script `setup_futuregateway.sh` calls the proper isntallation script relying on the kind of package manager found on the target host. At the moment the foreseen package managers are:

* brew - for MacOS X platforms
* apt - For Debian based platforms (Ubuntu 14.04 LTS) (not yet available)
* yum - For RedHat based platforms (CentOS 7) (not yet available)

The installation process foresees the following steps:

1. Identify the necessary components to install and the whole FutureGateway services topology. In case more hosts are involved in the setup process, please ensure to run the setup from a host able to connect via ssh ach node passwordlessly, properly exchanging SSH keys. Early phases of the setup will try to identify any missing mandatory configuration.
2. Modify the setup\_config.sh file configuring each FutureGateway service as designed in the previous step. Each FutureGateway component contains its own specific settings inside the setup\_config.sh script. Any FG user specified in the configurion file setup\_config.sh must be already present in its host system with passwordless sudo authorization as well as SSH key exchange with the installation node.
3. From the installation host, execute the script setup\_futuregateway.sh. The first time the setup procedure will install from Git all selected components, while further executions will try to upgrade the components and update its configurations accordingly with the values placed in the file setup\_config.sh

The script based installation procedure comes inside the repository named *PortalSetup* and it incudes cloud-init contextualization files to install FG on top of the EGI FedCloud.

## Using ansible playbooks

Different playbooks and roles are available for each specific FutureGateway compoenent; in particular:

* FutureGateway database
* APIServer front-end (fgAPIServer)
* APIServer (APIServerDaemon)
* Liferay62 *(not available yet)*
* Liferay7 *(not available yet)*
* [LiferayIAM](https://galaxy.ansible.com/indigo-dc/ansible-role-liferay-iam/) role is availble on ansible galaxy

### Usage
Before to install any node; verify that the account used to access the remote machine can connect via ssh as root without prompting for password. This is achieved properly configuring the ssh key file exchange.
To start the installation, configure first the `hosts` inventory file with the correct hostnames, then configure variable files under `vars/` directory and execute
```sh
ansible-playbook -i hosts <component name>
```
For instance to setup the database component just execute:
```sh
ansible-playbook -i hosts setupdb.yml
```

### Galaxy roles
The installation procedure relies on several ansible galaxy roles; you can install them by executing:
```sh
# All components
sudo ansible-galaxy install geerlingguy.git

# Database
sudo ansible-galaxy install geerlingguy.mysql

# fgAPIServer
sudo ansible-galaxy install geerlingguy.apache
sudo ansible-galaxy install bobbyrenwick.pip

# LiferayIAM
ansible-galaxy install indigo-dc.ansible-role-liferay-iam
```


# Old fashioned installation

The installation procedure is managed by a set of bash scripts operating at different levels. Low level scripts are in charge to install FutureGateway components targeting the different operating system. High level scripts manage low level scripts to install the system for a specific operating system.
Installation process may differ in case there exists a specific OS/Architecture hi-level isntallation script.

## Script structure

There are two kind of installation scripts; hi-level and low-level.
Hi level are mentioned to cover a specific OS/Linux distro; low level scripts are used by hi-level scripts. The principal aim of hi-level scripts is to deal with mandatory packages and prepare the right configuration for low level scripts; see file: 'setup_config.sh'.

There is no priority among low level setup scripts; except for setup_FGPortal.sh that must be the 1st to be executed. However the suggested priority is:

1. `setup_FGPortal.sh` - The main script, it takes care of Tomcat, Liferay and its SDK (optional), Other Development tools (ant, maven).
2. `setup_JSAGA.sh` - This script configure the environment to host JSAGA, including binariesm, libraries and taking care of the required UnlimitedJCEPolicy accordingly to the current JAVA version
3. `setup_GridEngine.sh` - This installs the Grid and Cloud Engine component (if requested)
4. `setup_OCCI.sh` - This is in charge to prepare the GSI environment (VOMS included) and the OCCI CLI interface. It may use the fed cloud installation or a manual setup (not really suggested, but necessary for CentOS7).
5. `setup_fgService.sh` - For OSes supporting the /etc/init.d service support this installs the service control script

All setup scripts have the same structure. Each installation step is managed by a dedicated bash function and each function can execute only once even running the setup script more times. This protection method is managed by a setup file named: '.fgSetup'.
The sequence of these function is managed by the scrip body at the bottom of the file in the form of an and-chain:

  `script_function_1 && script_function_2 && ... && script_function_n`

So that if one of the function fails the script terminates giving the opportunity to fix the issue and restart the installation.

## Using low level scripts
Architectures not targeted by hi-level scripts maybe installed as well, directly using low level scripts available at: https://github.com/indigo-dc/PortalSetup. These installation files can be used on all those OS platforms using as package management tool: yum, apt and brew; thus: EL OSes (i.e RedHat, CentOS, Debian, MacOSX and potentially Windows under Cygwin environment).
In order to use low level scritps, the user has to download from the git repository all files having the name 'setup\_\<component\>.sh'. Then the user can install the system just executing each script in the order:

1. git clone https://github.com/indigo-dc/PortalSetup && cd PortalSetup 
2. ./setup_FGPortal.sh - Install the core components of the system and preparing the necessary environment
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
# wget https://github.com/indigo-dc/PortalSetup/raw/master/Ubuntu_14.04/fgSetup.sh
# chmod +x fgSetup.sh
# cat /dev/zero | ssh-keygen -q -N ""
# cat /root/.ssh/id_rsa.pub >> /home/futuregateway/.ssh/authorized_keys
# echo "#FGSetup remove the following after installation" >> /etc/sudoers
# echo "ALL  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
# ./fgSetup.sh futuregateway futuregateway <your ssh port> $(cat /root/.ssh/id_rsa.pub)
```
Before to execute `fgSetup.sh` it is recommended to open it and verify inside its content, the default settings. Executing the last statement above, the installation procedure should start and it requires a while to complete.

## CentOS 7

Exactly like for Ubuntu, it is possible to install FutureGateway on CentOS7 machines following the same procedure of exchanging the SSH keys, be sure that futuregateway users is a passwordless sudo user during the installation and execute from remote or the same machine:

`./fgSetup.sh futuregateway futuregateway <your ssh port> $(cat $HOME/.ssh/id_rsa.pub)
`

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

### Suggested procedures
The installation scritps will instantiate the full FutureGateway environment extracting anything from GITHub, so that fresh installations will contain the latest available packages version or in alternative the version specified in the setup scripts. To know about the status or the end of the installation procedure, please check the output of the scripit.
Once finished the installation it is important to exit from any ssh connection active before the installation procedure and re-log again. During the re-connection, ssh will recognize a host identification change, then proceed to accept the new identity. In case the script have been executed from root it is enough to change the user with `su - futuregateway`.

In order to test FutureGateway REST APIs, several services should be started before; in particular:

1. The REST APIs [front-end][FGAPPDB]
2. The API Server Daemon [ServerDaemon][FGASRVD]

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
Detach with \<ctrl-a\>\<ctrl-d\>
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


## Executor Interfaces configuration

Executor interfaces may require to specify several settings

### ToscaIDC

This executor interface needs to configure several settings related to the PTV service.
The values have to be specified inside the web/WEB-INF/classes/it/infn/ct/ToscaIDC.properties file in particular the following values have to configured:

* `fgapisrv_ptvendpoint`: PTV hostname/address
* `fgapisrv_ptvuser`: PTV username
* `fgapisrv_ptvuser`: PTV password




