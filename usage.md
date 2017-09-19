# User guide

APIs are compliant with specification defined in: http://docs.fgapis.apiary.io/#
In particular this guide refers to specifications having version: v1.0.

## Tasks

### Submit tasks

Before submit any task, at least an application should be installed on the APIServer. Each application may have then one or several infrastructure defined where the application can execute. In APIServer terminology, the application may be a concrete running executable or an infrastructure.
Below example shoes how to create a new task for the baseline example application having id 1 `helloworld@localhost`. THis application just generate an output file and does not use any input file.

`curl -i -H "Content-Type: application/json" -X POST -d '{"application":"1","description":"helloworld@localhost test run","arguments": ["arg1","arg2","arg3"]}' http://localhost:8888/v1.0/tasks?user=brunor`

answer: `{"status": "WAITING", "application": 1, "date": "2015-10-08 16:22:51", "description": "helloworld@localhost test run", "output_files": ["hello.out", "hello.err"], "_links": [{"href": "/v1.0/tasks/1", "rel": "self"}, {"href": "/v1.0/tasks/1/input", "rel": "input"}], "user": "brunor", "input_files": ["hello.txt", "hello.sh"], "id": 1, "arguments": ["arg1", "arg2", "arg3"]}`

Please notice the use of filter `user=...`. Any APIServer REST call can be performed on behalf of a given user. Users maybe or not the one registered on the APIServer, however the user executing the command must have the special role `user_impersonate` in order to behave as a different user.

### Upload files and finalize task creation

The finalization, means that the task will be submitted; this happens as soon as all input_sandbox files have been sent

`curl -i -F "file[]=@hello.txt" -F "file[]=@hello.sh" http://localhost:8888/v1.0/tasks/1/input?user=brunor`

If there are no files to upload, the task will be submitted within the first curl command. Depending on the kind of action taken by the APIServer a different JSON output will be returned.

`$ curl -H "Authorization: Bearer TEST_TOKEN" -F "file[]=@input.txt" http://localhost:8888/v1.0/tasks/31/input
{
    "files": [
        "input.txt"
    ], 
    "message": "uploaded", 
    "task": "31", 
    "_links": [
        {
            "href": "/v1.0/tasks/31", 
            "rel": "task"
        }
    ], 
    "gestatus": "triggered"
}`

In this case the task is ready for the daemon processing.

`curl -H "Authorization: Bearer TEST_TOKEN" -F "file[]=@file.txt" http://localhost:8888/v1.0/tasks/32/input
{
    "files": [
        "file.txt"
    ], 
    "message": "uploaded", 
    "task": "32", 
    "gestatus": "waiting"
}`

In this case the task is still waiting for necessary input files.

Uploaded files can be retrieved using API calls getting its URL from the GET action on tasks endpoint.

`$ curl -H "Authorization: Bearer TEST_TOKEN" http://localhost:8888/v1.0/tasks/31
{
    "status": "SUBMIT", 
    "description": "tester application run", 
    "creation": "2017-09-19T08:04:22Z", 
    "iosandbox": "/tmp/258e35ac-9d11-11e7-a33d-fa163e876b0e", 
    "user": "00000000-0000-0000-0000-000000000000", 
    ...
    "input_files": [
        {
            "status": "READY", 
            "url": "file?path=%2Ftmp%2F258e35ac-9d11-11e7-a33d-fa163e876b0e&name=input.txt", 
            "name": "input.txt"
        }
    ], 
    "last_change": "2017-09-19T08:08:00Z"
}
$ curl -H "Authorization: Bearer TEST_TOKEN" "http://localhost:8888/v1.0/file?path=%2Ftmp%2F258e35ac-9d11-11e7-a33d-fa163e876b0e&name=input.txt"
###
### This is the input file
###
`

### Using the 2nd tester app 'sayhello.sh' (id:2)

This second example uses another sample application pre-configured after the installation. This application behaves like `helloworld@localhost` it has id: 2 and makes use of input and output files.
 
 1. `curl -i -H "Content-Type: application/json" -X POST -d '{"application":"2","description":"sayhello@csgfsdk test run", "arguments": ["\"I am saying hello!\""],  "output_files": [{"name":"sayhello.data"}], "input_files": [{"name":"sayhello.sh"},{"name":"sayhello.txt"}]}' http://localhost:8888/v1.0/tasks?user=brunor
 1 curl -i -H "Content-Type: application/json" -X POST -d '{"appliDELETEcation":"2","description":"sayhello@csgfsdk test run", "arguments": ["\"I am saying hello!\""],  "output_files": [{"name":"sayhello.data"}], "input_files": [{"name":"sayhello.sh"},{"name":"sayhello.txt"}]}' http://localhost:8888/v1.0/tasks?user=brunor`
 2. `curl -i -F "file[]=@sayhello.txt" -F "file[]=@sayhello.sh" http://localhost:8888/v1.0/tasks/14/input?user=brunor`

### The Grid and Cloud Engine (GridEngine)

In the case of the GridEngine executor interface, the original submission description JSON is  translated in a new format recognized by the GridEngine.

Follows a JSON understandable by Grid and Cloud Engine:

```
{
   "commonName": "brunor",
   "application": 106,
   "identifier": "Job - 1",
   "jobDescription": {
       "executable": "/bin/hostname",
       "output": "stdout.txt",
       "error": "stderr.txt"
   },
   "credential": {
       "type": "eTokenServer",
       "serverName": "etokenserver.ct.infn.it",
       "serverPort": "8082",
       "proxyId": "332576f78a4fe70a52048043e90cd11f",
       "vo": "gridit",
       "fqan": "gridit",
       "proxyRenewal": "true"
   },
   "infrastructure": {
       "resourceManagers": "wms://wms-4.dir.garr.it:7443/glite_wms_wmproxy_server",
         "informationSystem": "ldap://gridit-bdii-01.cnaf.infn.it:2170"
   },
 "userEmail": "mario.torrisi@ct.infn.it"
}
```
Translated to:
```
{
   "commonName": "test",
   "application": 106,
   "identifier": "Job0",
   "jobDescription": {
       "executable": "hostname",
       "output": "stdout.txt",
       "error": "stderr.txt"
   },
   "credential": {
     "type" : "UserPass",
     "username" : "glassfish",
     "password" : "im4Gine!"
   },
   "infrastructure": {
       "resourceManagers": "ssh://api.ct.infn.it"
   },
 "userEmail": "mario.torrisi@ct.infn.it"
}
{
   "commonName": "helloworld@localhost test run",
   "application": 10000, -- Refers to the GridEngine APIServer registered app
   "identifier": "task_id: 1",
   "jobDescription": {
       "executable": "hostname",
       "output": "stdout.txt",
       "error": "stderr.txt"
       "arguments": "-f"
   },
   "infrastructure": {
       "resourceManagers": "fork://localhost"
   }
}
```

JSON files above are located inside the task informative directory; see as_queue table field task_info in APIServer database.

#### Running test with GridEngine EI for accessing SSH 

1) Insert the task request

`curl -i -H "Content-Type: application/json" -X POST -d '{"application":"1","description":"helloworld@csgfsdk test run", "output_files": [ "stdout.txt","stderr.txt" ]}' http://localhost:8888/v1.0/tasks?user=brunor`

3) 2) Finalize task sending POST call (no uploads)

`curl -i -X POST http://localhost:8888/v1.0/tasks/1/input?user=brunor`

 This produced GridEngine understable JSON (application: 10000, means the APIServer app registered in UsersTracking database `GridOperations` table)

``{
 'infrastructure': {
    'resourceManagers': 'ssh://90.147.74.95'},
 'commonName': 'helloworld@csgfsdk test run',
 'application': '10000',
 'jobDescription': {
    'output': 'stdout.txt',
    'executable': '/bin/hostname',
    'arguments': '-f',
    'error': 'stderr.txt'},
 'credentials': {
    'username': 'jobtest',
    'password': 'Xvf56jZ751f'},
 'identifier': u'task_id: 1'
 }```

### Delete/Abort a task and all its references

The following commmand removes any reference to the task having id=2

`curl -i -X DELETE http://localhost:8888/v1.0/tasks/2?user=brunor`

Delete action is implemented by the APIServerDaemon which removes every task reference both from DB and file system except for the task table where the task entry will be kept and its status will be changed to 'PURGED'. Special DB cleaning procedures may delete PURGED records, avoiding the case that the last PURGED record is also the last record of the task table sorting it incrementally by task_id.

## runtime_data support

The system is able to associate run-time data to a running task
this feature offers to store any kind of information related to the
running task in the form of: (key-name, key-value) couple. Executor interfaces may use this field to store specific information related to the targeted infrastructure


## Applications

Below functions related to application management

List all applications:

`curl http://localhost:8888/v1.0/applications/`

List the app having id 2:

`curl http://localhost:8888/v1.0/applications/2`

Creation of an application:

`curl -i -H "Content-Type: application/json" -X POST -d '{"infrastructures": [1, {"description": "Tosca test at 90.147.170.168:32101","parameters": [{"name": "tosca_endpoint","value": "tosca://90.147.170.152:80/orchestrator/deployments"},{"name": "tosca_token","value": "11223344556677889900AABBCCDDEEFF"},{"name": "tosca_template","value": "tosca_template.yaml"},{"name": "tosca_parameters","value": "wait_ms=30000&max_waits=30"}],"enabled": true,"virtual": false,"name": "tosca_Test@BA"}],"parameters": [{"name": "target_executor","value": "SimpleTosca","description": ""},{"name": "jobdesc_executable","value": "tosca_test.sh","description": ""},{"name": "jobdesc_output","value": "stdout.txt","description": ""},{"name": "jobdesc_error","value": "stderr.txt","description": ""}],"enabled": true,"input_files": [{"override": false,"path": "/home/futuregateway/FutureGateway/fgAPIServer/apps/toscaTest","name": "tosca_template.yaml"},{"override": false,"path": "/home/futuregateway/FutureGateway/fgAPIServer/apps/toscaTest","name": "tosca_test.sh"}],"name": "hostname@tosca(1)","description": "hostname tester application on tosca by setup(1)"}' http://localhost:8888/v1.0/applications?user=brunor`

Infrastructure JSON section, foresees both the possibility to use a generic infrastructure or an application level ifnrastructures. In the first case it  is necessary to specify the `id` of the infrastructure 
Please be aware that application installation API does not take care of FutureGateway defined uers groups and values. In order to allow users to run this application it is necessary to allow at least one group to run this application, inserting a new record in table fg_group_apps: insert into fg_group_apps values ( \<group_id\>, \<app_id\>, now());.

Inside `$FGLOCATION/fgAPIServer/`apps directory, there are several applications that can be used as template to generate new applications. Once selected the most appropriate application to use, just make a full copy of the application direcotry, provide to the new directory the application name and go inside of it. In the application directory there are two files: setup\_app.sh and stress\_test.sh. Modify the first file to setup properly the new application, use the second file to execute the new application on the system. The second script can be executed only after the setup\_app.sh script has been executed. The setup_app.sh script contains several configuration settings and it can be executed specifying as argument a session token (with no 'Authentication: Bearer' string).

After the applciation is created, it is important to assign this application to a group, so that their members can perform any allowed activity on it (See advanced operations).

Input files

The original API specifications did not have the concept of ‘application level’ files because not foreseen by the original API specification. The old fashioned way to specify default application files was the use of tag ‘input_files’ inside the application installation JSON. The ‘input_files’ can or cannot overridden by the user during task submission accordingly to a given flag. 
API specification have been modified in order to cover ‘application’ level files, considering a new JSON tag in application installation named ‘files’. Files accepts a list of filenames and then a new API endpoint named `application/input/<app_id>`  with POST method upload these files. During the task submission these files will be included in the task file sandbox automatically and the user cannot modify/access them. In order to do not compromise the backward compatibility the fgAPIServer implements both tags ‘input_files’ and ‘files’. 

Using input_files:

```
{..., "input_files": [{"override": false,
                       "path": "<path to file>",
                       "name": "file name"}, ...}, ...}
```

The override flag when true avoids to change the file file content during the task submission phase.
The path have to point to a directory containing the file name specified in the name field.
This method to specify input files only work with API server filesystem.
This method is useful to write application installation scripts.

Using files:

```
{... "files": [ "file1", "file2", ...], ...}' 
```

Then upload each file content with:

```
curl  -H "Authorization: Bearer XXX" -F "file[]=@file1 -F "file[]=@file2 ..."  http://localhost:8888/v10/applications/1/input
```

View uploaded files:

```curl -H "Content-Type: application/json" http://localhost:8888/v1.0/applications/input```

This method is suggested when installing applications using external GUIs because files can be uploaded from anywhere.
Specified input files cannot be changed by the user during the submission phase; they are threated as 'override == true' of the input_file case.

Delete

`curl -i -H "Content-Type: application/json" -X DELETE`

Change an application:

```curl -H "Content-Type: application/json" -H "Authorization: Bearer TKN"  -X PUT -d '{"files": ["tosca_template.yaml", "tosca_test.sh"],"name": "hostname@toscaIDC","parameters": [{"name": "target_executor","value": "ToscaIDC","description": ""}, {"name": "jobdesc_executable","value": "tosca_test.sh","description": "unused"},{"name": "jobdesc_output", "value": "stdout.txt","description": "unused"},{"name": "jobdesc_error", "value": "stderr.txt", "description": "unused"}], "outcome": "JOB","enabled": true,"id": "14","infrastructures": [4],"description": "hostname tester application on toscaIDC"}' http://localhost:8888/v1.0/applications/14```

Application change supports both files and input_files. Using files the application will not change filenames already present in the application files list and removes files no more present in the new passed list. The use of the field input_files behaved in the same way of files except that uses path and override additional fields. 

Infrastructures can be specified using registered Ids or using an explicit json declaration like in infrastructure creation. All specified infrastructure ids will be checked against the existing ids. New ids will be associated to the application while ids no more present in the passed json array will be removed. Specifiying explicitly an infrastrcucture description; a new infrastructure will be created and registered with the application; for instance:

```curl -H "Content-Type: application/json" -H "Authorization: Bearer TKN"  -X PUT -d '{"files": ["tosca_template.yaml", "tosca_test.sh"],"name": "hostname@toscaIDC","parameters": [{"name": "target_executor","value": "ToscaIDC","description": ""}, {"name": "jobdesc_executable","value": "tosca_test.sh","description": "unused"},{"name": "jobdesc_output", "value": "stdout.txt","description": "unused"},{"name": "jobdesc_error", "value": "stderr.txt", "description": "unused"}], "outcome": "JOB","enabled": true,"id": "14","infrastructures": [4, { "name": "New infra test", "parameters": [ { "name": "jobservice", "value": "ssh://localhost", "description": "local SSH host" }, { "name": "username", "value": "jobtest" }], "description": "New ssh infra", "enabled": true, "virtual": false }],"description": "hostname tester application on toscaIDC"}' http://localhost:8888/v1.0/applications/14``` 

With the explicit infrastructure description in the application change, it is possible to specify application specific values to already existing infrastructures.

### REST APIs can get/set runtime_data 

This is included in the json response after a GET: tasks/<task_id> inside 'runtime_data' json section specifying an input json in the form:
```
{ "runtime_data" : [ { "data_name":  "name"
                       ,"data_value": "value"
                       ,"data_desc": "description of the value"
                      }, ... ]
```

REST APIs can insert/change runtime values with PATCH: `tasks/<task_id>`
 
View task detail (watch runtime_data)

`curl -i http://localhost:8888/v1.0/tasks/43?user=brunor`

Insert a new data: `http://localhost:8888/v1.0/applications/2`

`curl -i -H "Content-Type: application/json" -X PATCH -d '{"runtime_data": [ { "data_name": "test_data", "data_value": "test_value", "data_desc": "test description value"} ]}' http://localhost:8888/v1.0/tasks/43?user=brunor`

### PATCH on task status

The PATCH call on tasks endpoint has been enriched in order to support a task status change. Executor interfaces may recognize this change forcing the requested status of the task into the distributed infrastructure. In ToscaIDC executor interface, it is posstible to request status `'CANCELLED'` thus will enforce to release the resource associtated to the task to be released by the TOSCA orchestrator.
To specify a status change use:

`curl -i -H "Content-Type: application/json" -X PATCH -d '{"status": "CANCELLED" }' http://localhost:8888/v1.0/tasks/43?user=brunor`



# Change a data (notice that desc is not specified anymore
`curl -i -H "Content-Type: application/json" -X PATCH -d '{"runtime_data": [ { "data_name": "test_data", "data_value": "new_test_value"} ]}' http://localhost:8888/v1.0/tasks/43?user=brunor`

# Authentication and Authorization

## Baseline AuthN/Z token managerment

This is the way futuregateway manages authorization and authentication as default. This method can be switched off, modified or substituded by other AuthN/Z systems

1) Obtain a login token (not really mandatory step; see step 2)

	There exists a small code called mklogtoken.py

	`python ./mklogtoken.py
	mwDn8D9ev4ScUWLqVNiXo40B4w/jrp0vX7CjEwC52Lg9IQHhI4O0AE74afTbpreS`

	This code requires three inputs
	`key = "0123456789ABCDEF”` The key values used to encrypt the user credentials
	`username = “test”`  The username
	`password = “test”`  Its password

	Example (using values above):
    ```
	Macbook@RicMac:~/Documents/fgapiserver$ python ./mklogtoken.py	
	mwDn8D9ev4ScUWLqVNiXo40B4w/jrp0vX7CjEwC52Lg9IQHhI4O0AE74afTbpreS
	````
	This string simply encripts username,password and timestamp infrormation 

	This code could be included inside a login web form where user provides his credentials or even on the mobile app; what is really important is the key value used to encrypt the message. The key must be the same also in the front-end. At the moment placed in the configuration file $FGLOCATION/fgAPIServer/fgapiserver.conf (fgapiserver_secret field).

 2 ) Obtain a session token using the logtoken via POST

	```Macbook@RicMac:~$ curl -i -H "Content-Type: application/json" -H "Authorization: Bearer mwDn8D9ev4ScUWLqVNiXo40B4w/jrp0vX7CjEwC52Lg9IQHhI4O0AE74afTbpreS" -X POST http://localhost:8888/v1.0/auth
	HTTP/1.0 200 OK
	Content-type: application/json
	Content-Length: 155
	Access-Control-Allow-Origin: *
	Access-Control-Allow-Headers: Content-Type,Authorization
	Access-Control-Allow-Methods: GET,PUT,POST,DELETE,PATCH
	Access-Control-Allow-Credentials: true
	Server: FutureGateway API Server v1.0
	Date: Thu, 19 May 2016 14:04:35 GMT
	
	{
	    "token": "9dd5e25c-1dca-11e6-91f8-d5a4b6a30f62",
	    "_links": [
	        {
	            "href": "/auth",
	            "rel": "self"
	        }
	    ]
	}
	```
2.a) It is possible to obtain a session tokens direcrtly from /auth call providing credentials as parameters (operation suggested only if the front is accessible via https.

## Portal Token Validation 

PTV can be enabled by a configuration flag, in this case session tokens are verified against an existing portal which answers to a given endpoint. A username and password is necessary to contact the portal endpoint and an https connection should be adopted.
The portal returns a flag telling if the user is allowed or not and optionally further information such the user name and its group in the portal so that this information can be used  to map the portal user with a futuregateway user.
While processing APIs using tokens related to users not yet registered in the API Server, the new user will be automatically registered using as reference the received subject field.

## Debug with Liferay using Shibboleth
Once connected get from the browser shibboleth session and pass it as cookie; for instance:
`curl --cookie "_shibsession_64656661756c7468747470733a2f2f7367772e696e6469676f2d64617461636c6f75642e65752f73686962626f6c657468=_9dcb88c4f6b46005a1f2403079dd738f" -v https://sgw.indigo-datacloud.eu/apis/v1.0/tasks?user=brunor`

Another option is:

 1. Once logged place the section cookie in a 'cookie.txt' file
 2. In curl commands specify option: -b cookie.txt

## Infrastructures
Any installed application on the APIServer may have defined one or more infrastructures where it can execute. 
The current version of the software can support both generic infrastructures shareable across different applications as well as application level infrastructures.

### Generic infrastructures
To install a generic ifnrastructure, have been implemented the Infrastructure APIS in accordance to the Futuregateway specifications: http://docs.fgapis.apiary.io/#

#### Infrastructure creation:

```curl -i -H "Content-Type: application/json" -X POST -d '{ "name": “Infra test“, "parameters": [ { "name": "jobservice", "value": "ssh://localhost" }, { "name": "username", "value": "jobtest" }, { "name": "password", "value": "Xvf56jZ751f" } ], "date": "2017-01-10T16:59:10Z", "description": "ansshinfra", "enabled": true, "virtual": false }' http://localhost:8888/v1.0/infrastructures```

#### View infrastructure:

```curl -H "Authorization: Bearer TOKEN" http://localhost:8888/v1.0/infrastructures/2```

### Application level infrastructures
It is still possible to install infrastructures when installing new applications specifying them in the application creation rest call instead to pass the infrastructure array of `<id>`s as reported in the specifications. In this case the user has to pass an array of defined infrastructures. This solution has been intentionally maintained because by previous experiences in most of the cases each application requires its own specific settings. However the infrastructure sharing among application is still possible using infrastructures API endpoints. It is still possible to define applications directly operating ad database level, for more information about this opportunity, please have a look on the 'Advanced operation' section.

#### Change infrastructure:

```curl -H "Content-Type: application/json" -H "Authorization: Bearer TKN"  -X PUT -d '{"name": "Infra test (changed)", "enabled": false, "id": 6, "virtual": true, "description": "ansshifnra (changed)", "parameters": [{"name": "jobservice", "value": "ssh://fgtest", "description": "fgtest ssh hots"} ] }' http://localhost:8888/v1.0/infrastructures/6```


## Executor Interfaces

Executor interfaces (EI)s are the way the APIServer uses to reach any kind of distributed computing resource through the use of any possible middleware component. EIs may deal with very flexible system like JSAGA and its adaptors like in GridEngine and SimpleTosca executor interfaces. In other cases EIs may use APIs or even CLI commands to manage the distributed infrastructure like in the case of the ToscaIDC which deals directly with TOSCA orchestrator REST API calls. Below sections describe how to configure these interfaces, in particular how to setup properly `infrastructure_parameters` values while describing a new APIServer application.


### GridEngine
The GridEngine executor interface target any distributed infrastructure that the Grid and Cloud component can address. The most popular and supported ones are: SSH, EMI/gLIte and rOCCI.

* SSH Configuration
. `jobservice`: ssh://\<ssh_host\>:\<ssh_port\>
. `username`: the ssh username
. `password`: the ssh password (clear text)
Omitting the `password` field or placing an empty string the GridEngine will attempt to establish the SSH connection using the .ssh private key file contained in the tomcat server home dir running the APIServerDaemon web application.

* EMI/gLite Configuration
. `jobservice`: wms endpoint
. `bdii`: informatio provider endpoint
. `eToken_host` eTokenServer hostnmae
. `eToken_port` eTokenServer listening port
. `eToken_id`: robot proxy identifier
. `voms`: voms name
. `voms_role`: the role associated to the voms
. `rfc_proxy`: flag to request or not an RFC proxy

* rOCCI Configuration
. `jobservice`: rOCCI endpoint
. `os_tpl`: OS template name
. `resource_tpl`: resource template name
. `attributes_title`: VM instance name
. `eToken_host` eTokenServer hostnmae
. `eToken_port` eTokenServer listening port
. `eToken_id`: robot proxy identifier
. `voms`: voms name
. `voms_role`: the role associated to the voms
. `rfc_proxy`: flag to request or not an RFC proxy

### SimpleTosca
SimpleTosca is a deprecated EI that has been used in early phases of FG development. This EI has been designed to cover explicitly the Molecular Dynamics (MD) use case and it is no longer used. The description of this EI is provided just for completeness.

* Configuration
. `tosca_endpoint`: The TOSCA endpoint
. `tosca_token`:  Token for access TOSCA endpoint (unused)
. `tosca_template`: The yaml file name describing the TOSCA resource
. `tosca_parameters`: Parameters supported by jsaga-tosca-adaptor

### ToscaIDC
The ToscaIDC executor interface deprecates the SimpleTosca and it may cover any possibility offered by the TOSCA orchestrator since it is possible to send any possible yaml file containing any possible yaml input parameter.

* Configuration
. `tosca_endpoint`: The TOSCA endpoint
. `tosca_template`: The yaml file name describing the TOSCA resource
. `tosca_parameters`: TOSCA parameters (params=\<json file containing yaml input parameters\<)

Inside directory $FGLOCATION/apps there are several sample application making use of different EIs. Any user may use one of these tester applications to generate as a template its own specific application use case. For ToscaIDC there exists a sample application named: ToscaIDCTest. This application is configured to run as default withting the fgportal_ptv.py script which simulates the TOSCA orchestrator as well as a PTV endpoint. 

## Advanced operations

The APIServer manages users, groups and roles however API specs do not take in account these entities. For instance when a new application is installed it is necessary to enable a group to grant execution rights to the new application. All these operations not foreseen by the API specifications must be performed managing directly the database. For this reason it is important for developers and administrators to know the APIServer database schema.

### Application tables

These tables are collecting the information related to APIServer installed applications

* `application`: Store generic application information
```
id:          record identifier
name:        application name
description: application description
outcome:     explains the kind of application: (JOB|INFRASTRUCTURE)
creation:    application installation timestamp
enabled:     enable/disable flag
```
* `application_file`: Store input files required by the application
```
app_id:   Applciation record identifier
file_id:  File index number
file:     File name
path:     Path of the file
override: Force to use this file even if the user specify an alternative file
```
* `application_parameter`: 
```
app_id:   Application record identifier
param_id: Parameter index number
pname:    Name of the parameter
pvalue:   Value of the parameter
pdesc:    Parameter description
```

### Infrastructure tables

* `infrastructure`: This table should have a different name like `application_infrastructure`, since the primary key of this table consists of the couple (application id, infrastructure id). The infrormation about infrastructure is kept at application level in order to facilitate infrastructure customization for a specific application.
This table contains just infrastructure generic information and several flags like enabling the infrastructure or specify if the infrastructure is a virtual entity (something to be instantiated yet).
```
id          Infrastructure record identifier
app_id      Application record identifier where this infrastructure belongs
name        Name of this infrastructure
description Infrastructure description
creation    Creation timestamp
enabled     Enable/Disable flag
vinfra      True if this is a virtual infrastructure
```

Please notice that infrastructure sharing among different applications can be defined playing around the primary key (application id, infrastructure id). However the use of infrastructure sharing is highly not recommended.

* `infrastructure_parameter`: In the form of (key name, key value) couples this table contains application infrastructure specific settings.
```
infra_id: Infrastructure record identifier
param_id: Parameter index number
pname:    Parameter name
pvalue:   Parameter value
pdesc:    Parameter description
```

* `infrastructure_parameter`: In the form of (key name, key value) couples this table contains application infrastructure specific settings.
```
infra_id: Infrastructure record identifier
param_id: Parameter index number
pname:    Parameter name
pvalue:   Parameter value
pdesc:    Parameter description
```

* `infrastructure_task`: The functionality related to this table is not yet supported and it is related to the use of virtual infrastructures. Whenever a virtual infrastructure instance will be created by a task this table will link the task requiring the infrastructure with an infrastructure dinamically created by an 'infrastructure' application (see outcome field in applciation table).
```
infra_id: Infrastructure record identifier
task_id:  Task record identifier
app_id:   Application id requesting this infrastructure
creation: Infrastructure creation time
```

## Task related tables

* task: Generic information about task activity is registered in this table.
```
id:          Task record identifier
creation:    Task creation timestamp
last_change: Last change timestamp done on the task
app_id:      Application record identifier related to this task
description: Task description
status:      Status of the task
iosandbox:   Directory path that store any file related to this task
user:        User associated to this task (may differ from token recognized user)
```
* task_arguments: Each task may spevify different arguments stored in the form o
```
task_id:  Task record identifier
arg_id:   Argument index number
argument: Argument value
```
* task_input_file: Task specific input files
```
task_id: Task record identifier 
file_id: File index number
file: Name of the file
path: Directory path where file is located
```
* task_output_file: Task specific output files
```
task_id: Task record identifier 
file_id: File index number
file: Name of the file
path: Directory path where file is located
```

* runtime_data: This table is related to tasks as well, it stores in the form of (key name, key value) any information related to the task.
```
task_id:     Task record identifier
data_id:     Runtime data index number
data_name:   Runtime data name
data_value:  Runtime data value
data_desc:   Runtime data description
data_proto:  Access protocol to access the data
data_type:   Type of data (bynary, json, plain/text, ...)
creation:    Creation time of the data
last_change: Last change made on the record; data values may be overwritten
```

## Queue table

* as_queue: The APIServer queue table keeps track of any command sent to the APIServer Daemon. This table normally stores activities to be performed on tasks: creation, deletion, etc. This table  can be used also to extend the APIServer daemon functionalities (See for instance the executor interfaces). The queue is filled by the front-end (fgAPIServer) and consumed buy the queue polling daemon (APIServerDaemon).
```
task_id:       Task record identifier
target_id:     Target executor interface record identifier
target:        Executor interface name
action:        Command to execute
status:        Status of the command
target_status: Status of the command in the targeted infrastructure
retry:         Current number of command retry
creation:      Record creation timestamp
last_change:   Timestamp of the last hange done on the record
check_ts:      Timestamp of the last check done on the record
action_info:   Directory path assigned to the task
```

## Users, Groups and Roles
The APIServer manages its own set of users, groups and roles. The baseline setup creates few users examples providing each a different set of privileges. Different sets of privileges may be assigned to groups and groups can be assigned to users.

* fg_user: Stores generic information about the APIServer registered user.
```
id         User record identifier
name       Name (nick) of the user
password   Shadowed user password 
first_name First name of the user
last_name  Last name of the user
institute  User insitution
mail       User email address
creation   User registration timestamp
modified   User record change timestamp
```

* fg_group: Stores generic information about the stored groups.
```
id       Group record identifier
name     Name of the group
creation Group creation timestamp
modified Group record change timestamp
```

* fg_user_group: Link available users with available groups
```
user_id  User record identifier
group_id Group record identifier
creation Link creation timestamp
```

* fg_group_apps: Associate an application to an existing group
```
group_id: Group record identifier
app_id:   Application record identifier
creation: Record creation timestamp
```

* fg_group_role: Store any role associated to the groups
```
group_id: Group record identifier
role_id:  Role record identifier
creation: Record creation timestamp 
```

* fg_role: Store any possible role foreseen by the APIServer (Not all roles defined by the baseline setup are supported in current implementation).
```
id:       Role record identifier
name:     Name of the role
creation: Record creation timestamp
modified: Record change timestamp
```

Current defined roles are:

* `app_install`: The user can install applications       
* `app_change`: The user chan perform changes to the applciations
* `app_delete`:  The user can delete applications
* `app_view`:  The user can see details of an application
* `app_run`: The user can run the application          
* `infra_add`: The user can add an infrastructure        
* `infra_change`:  The user chang make changes to an infrastructure
* `infra_delete`: The user can delete an infrastructure
* `infra_view`:  The user can see details of an infrastructure
* `infra_attach`: The user can attach an infrastructure to an application/task
* `infra_detach`:  The user can detach an infrastructure to an application/task
* `task_delete`: The user can delete a task
* `task_view`: The user can see task details        
* `task_userdata`: The user can access to task userdata    
* `user_add`: The user can add other users
* `user_del`:  The user can remove other users
* `user_change`: The user can change details of another user
* `group_change`: The user can change group settings
* `role_change`:  The user can change roles on groups
* `user_impersonate`:  The user can impersonate a different user
* `group_impersonate`: The user can impersonate a different group

*  fg_token: Store associations between APIServer users and token received by the front-end (fgAPIServer).
```
token:    Token value
user_id:  User record identifier associated to the token
creation: Record creation timestamp
expiry:   Token expiry timestamp  
```

## DB Schema

* db_patches: This table is used by the database schema updater script available under `$FGLOCATION/fgAPIServer/db_patches` directory. It stores all patches applied to the original baseline database schem installation.
```
id:      DB Patch record identifier
version: DB schema version reached by the patch
name:    Name of the patch
file:    File name that applied the patch
applied: Timestamp when the patch has been applied
```


### Executor Interfaces DB
Executor interface may have its own specific table or (as in the case of the Grid and Cloud engine) a whole database. The responsability to deal with these tables is in charge of the specific Executor Interface.

* SimpleTosca/ToscaIDC: These Executor interfaces are using respectively the tables: simple_tosca and tosca_idc. Both tables have the same structure:
```
id:           Record identifier
task_id:      Associated task record identifier
tosca_id:     TOSCA UUID value
tosca_status: Status reported by the orchestrator
creation:     Record creation timestamp
last_change:  Record change timestamp  
```

* UsersTrackingDatabase: The UsersTrackingDatabase is used by the Grid and Cloud Engine component to keep track of each activity performed by this system. This database has been developed to fullfil the EGI Traceability policies defined for scientific gateways. The Grid and Cloud Engine was the core component of the Catania Science Gateway Framework CSGF from where the FutureGateway takes its orings. The GridEngine executor interface deals with the Grid and Cloud Engine component in order to target the following kind of distributed infrastructures: SSH, EMI/gLite, rOCCI. All of them are reached instructing properly the JSAGA component through the use of the corresponding JSAGA adaptor. This involves that applications targeting the GridEngine executor interface may run on all of those infrastructure just filling up the `infrastructure` and `infrastructure_parameters_tables`. This kind  of configuration is visibile in the baseline setup with the `sayhello` application. When submitting a task linked to many infrastructures, only one of them will be selected using a random strategy. This behavior may be changed in the future with more sophisticated algorithms depending for instance on the loud charge of each targeted infrastructures.

## fgTools

APIServer software suite offers the fgTools repository where can be located any utility and helper tool. At the moment only two utilities are available:

* `pushProxy`: This is an utility developed at the very ealry developments on TOSCA orchestrator. The utility has to be configured inside a cron job in order to timely send to a given endpoint a robotProxy extracted from the eTokenServer host.

* `updateCode`: This utility helps FG developers to update source packages remotely. It allows to send local modified APIServerDaemon sources, recompiling and installing them on the remote service. It provides the same capability for other sources like adaptors and the front-end. It also provide options to send and receive files from/to the remote FG machine.
Below the help message:
```
usage: updateCode  -h <ip|host>                  # FutureGateway host
                    [-p <ssh port>] (default 22) # FutureGateway ssh port
                    [-c <fgAPIServer             # code: APIServer front-end
                        |APIServerDaemon         # code: APIServerDaemon
                        |jsaga-adaptor-rocci>    # code: rOCCI adaptor
                        |jsaga-adaptor-tosca>]   # code: tosca adaptor
                    [ -f <file/directory path>   # Upload Local file/dir 
                    [-t <destination dir> ]]     # \$FGLOCATION/<file/dir>
                    [-u <file/directory path>]   # Download file/dir from remote
                    [-a]                         # Update all components
                    [-s]                         # Start futuregateway service
```
This script copies the specified source package into specified remote host
using ssh connection, for this reason it is recommended to exchange the
ssh keys between source and destination hosts
By default the script stops the futuregateway service leaving it not running
unless the the option -s is specified


	



