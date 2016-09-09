# Installation

FutureGateway can be installed on top of the following environments:
* Enterprise Linux 6/7
* Debian (Ubuntu)
* MacOSX
The installation procedure is managed by a set of bash scripts operating at different levels. Low level scripts are in charge to install FutureGateway components targeting the different operating system. High level scripts manage low level scripts to install the system for a specific operating system.


Installation process may differ in case there exists a specific OS/Architecture hi-level isntallation script.

## Using low level scripts
Architectures not targeted by hi-level scripts maybe installed as well, directly using low level scripts. These installation files can be used on all those OS platforms using as package management tools: yum, apt and brew; thus: EL OSes (i.e RedHat, CentOS, Debian, MacOSX and potentially Windows under Cygwin environment).
In order to use low level scritps, the user has to download from the git repository all files having the name 'setup\_\<component\>.sh'. Then the user can install the system just executing each script in the order:

1) ./setup_FGPortal.sh - Install the core components of the system and preparing the necessary environment
2) ./setup_JSAGA.sh - Provides a complete JSAGA installation with binaries and libraries
3) ./setup_GridEngine.sh - Install the Grid&Cloud Engine (necessary to target SSH, EMI/gLite, rOCCI infrastructures) 
4) ./setup_OCCI.sh - Install the OCCI CLI with GSI necessary packages
5) ./setup_FGService.sh - Execute only in case you want to run FutureGateway as a service

Before execute any 'setup\_\<component\>.sh' script; the user has to configure the file setup\_config.sh file to override default configuration settings written at the top of each setup file.
Once installation files have been executed; it is necessary to log-out and login again; so that the new environment will be ready and the two APIServer components (fgAPIServer and APIServerDaemon) can be installed from sources with the following steps:

1) Download fgAPIServer files from gitHub in the directory: $FGLOCATION/fgAPIServer
2) Execute fgapiserver.py script inside a dedicated screen section or confiure wsgi to run it
3) Download APIServerDaemon files from gitHub in the directory: $FGLOCATION/APIServerDaemon
4) Go inside the APIServerDaemon directory and compile java code with: ant all
5) Copy the generated war file into $CATALINA_HOME/webapps directory and check on $CATALINA_HOME/logs/catalina.out file that the installation is successful

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
# wget https://github.com/FutureGateway/PortalSetup/raw/master/Ubuntui_14.04/fgSetup.sh
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
The virtual appliance is based on an Ubuntu-server 14.04 and it requires a specific cloud-init' user-data file in order to setup it properly.
The principal aim of the FutureGateway virtual appliance is to allow Science Gateway application developers to make practice with FutureGateway REST APIs without takin care of the whole system intallation.

Below the commands to correctly instantiate the VM:

`OCCI_RES=$(occi -e $OCCI_ENDPOINT --auth x509 --user-cred $USER_CRED --voms $VOMS --action create --resource compute --mixin os_tpl#$OS_TPL --mixin resource_tpl#$RESOURCE_TPL --attribute occi.core.title="fgocci2" --context user_data="file://$HOME/userdata.txt"); echo "Resourcce: $OCCI_RES"`

The contextualization file `userdata.txt` has to be customized before to execute the `occi` command line. In particular the user must provide its own ssh public key under cloud-init key `ssh-authorized-keys`. Another point to customize is inside the `write_files` directive for the file `/root/installFG.sh`. The following sed commands must be configured:

`sed s/TOMCATUSR=\"tomcat\"/TOMCATUSR=\"<tomcat_admin_user>\"/`
`sed s/TOMCATPAS=\"tomcat\"/TOMCATUSR=\"<tomcat_admin_password>\"/`

replace text inside the `<...>` brackets with your preferred Tomcat service admin username and password.
Further and more sofisticated customizations could be done in the same fashion mofifying the downloaded script `fgSetup.sh`

In case it is needed to assign a public IP to the given resource, execute:

`occi --endpoint $OCCI_ENDPOINT --auth x509 --user-cred $USER_CRED --voms --action link --resource $OCCI_RES --link /network/public`

# Suggested procedures
The installation scritps will instantiate the full FutureGateway environment extracting anything from GITHub, so that fresh installations will contain the latest available packages version. To know about the status or the end of the installation procedure, please check the output of the scripit.
Once finished the installation it is important to exit from any ssh connection active before the installation procedure and re-log again. During the re-connection, ssh will recognize a host identification change, then proceed to accept the new identity. In case the script have been executed from root it is enough to change the user with `su - futuregateway`.

In order to test FutureGateway REST APIs, several services should be started before; in particular:

1. The REST APIs [front-end][FGAPPDB]
2. The API [ServerDaemon][FGASRVD]

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

## APIServer Daemon (APIServerDaemon)
The API Server Daemon conists of a web application, so that it is necessary to startup the application server (Tomcat). The virtual appliance is already configured to install and execute the daemon during the application server startup.
To startup the application server you may use the standard scripts provided with Tomcat or you may use the 'start\_tomcat' utility:

* Startup application server:
`start_tomcat`. To manage daemon activity you can use the Tomcat manager front-end with `http://<VM_IP>:8080/manager` (default credentials are tomcat/tomcat).To stop Tomcat you can use `stop_tomcat` then please verify its java process with `ps -ef | grep tomcat | grep java` if the process still perist you may use '`killjava` command.

* Monitor the APIServer daemon app server activity:
`tail -f $CATALINA_HOME/logs/catalina.out`
It is important during development phases to constatly monitor the APIServer daemon activity, to accomplish that it is enough to have a look inside the application server log file.

* Monitor the APIServer daemon activity:
`tail -f $FGLOCATION/apache-tomcat-8.0.26/webapps/APIServerDaemon/WEB-INF/logs/APIServerDaemon.log`

* Monitor the GridEngine activity:
t.b.d.

## Security considerations
Although the VM has been configured to limit hackers exposure, it is warmly suggested to comply with the EGI FedCloud [directives][EGIFCDR]

[FGAPIFE]: <https://github.com/FutureGateway/fgAPIServer>
[FGASRVD]: <https://github.com/FutureGateway/APIServerDaemon>
[EGIFCDR]: <https://wiki.egi.eu/wiki/Virtual_Machine_Image_Endorsement#Hardening_guidelines>
