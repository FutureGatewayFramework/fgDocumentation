# Usage

APIs are compliant with specification defined in: http://docs.fgapis.apiary.io/#
In particular this guide refers to specifications having version: v1.0.

## Tasks

### Submit tasks

Before submit any task, at least an application should be installed on the APIServer. Each application may have then one or several infrastructure defined where the application can execute. In APIServer terminology, the application may be a concrete running executable or an infrastructure.
Below example shoes how to create a new task for the baseline example application having id 1 `helloworld@localhost`. THis application just generate an output file and does not use any input file.

`curl -i -H "Content-Type: application/json" -X POST -d '{"application":"1","description":"helloworld@localhost test run","arguments": ["arg1","arg2","arg3"]}' http://localhost:8888/v1.0/tasks?user=brunor`

answer: `{"status": "WAITING", "application": 1, "date": "2015-10-08 16:22:51", "description": "helloworld@localhost test run", "output_files": ["hello.out", "hello.err"], "_links": [{"href": "/v1.0/tasks/1", "rel": "self"}, {"href": "/v1.0/tasks/1/input", "rel": "input"}], "user": "brunor", "input_files": ["hello.txt", "hello.sh"], "id": 1, "arguments": ["arg1", "arg2", "arg3"]}`

Please notice the use of filter `user=...`. Any APIServer REST call can be performed on behalf of a given user. Users maybe or not the one registered on the APIServer, however the user executing the command must have a special role in order to behave as a different user.

### Upload files and finalize task creation

The finalization, means that the task will be submitted; this happens as soon as all input_sandbox files have been sent

`curl -i -F "file[]=@hello.txt" -F "file[]=@hello.sh" http://localhost:8888/v1.0/tasks/1/input?user=brunor`

If there are no files to upload, the task will be submitted within the first curl command. Depending on the kind of action taken by the APIServer a different JSON output will be returned.

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


## runtime_data support

The system is able to associate run-time data to a running task
this feature offers to store any kind of information related to the
running task in the form of: (key-name, key-value) couple


## Applications

Below functions related to application management

List all applications

`curl http://localhost:8888/v1.0/applications/`

List the app having id 2

`curl http://localhost:8888/v1.0/applications/2`

Creation of an application

`curl -i -H "Content-Type: application/json" -X POST -d '{"infrastructures": [{"description": "Tosca test at 90.147.170.168:32101","parameters": [{"name": "tosca_endpoint","value": "tosca://90.147.170.152:80/orchestrator/deployments"},{"name": "tosca_token","value": "11223344556677889900AABBCCDDEEFF"},{"name": "tosca_template","value": "tosca_template.yaml"},{"name": "tosca_parameters","value": "wait_ms=30000&max_waits=30"}],"enabled": true,"virtual": false,"name": "tosca_Test@BA"}],"parameters": [{"name": "target_executor","value": "SimpleTosca","description": ""},{"name": "jobdesc_executable","value": "tosca_test.sh","description": ""},{"name": "jobdesc_output","value": "stdout.txt","description": ""},{"name": "jobdesc_error","value": "stderr.txt","description": ""}],"enabled": true,"input_files": [{"override": false,"path": "/home/futuregateway/FutureGateway/fgAPIServer/apps/toscaTest","name": "tosca_template.yaml"},{"override": false,"path": "/home/futuregateway/FutureGateway/fgAPIServer/apps/toscaTest","name": "tosca_test.sh"}],"name": "hostname@tosca(1)","description": "hostname tester application on tosca by setup(1)"}' http://localhost:8888/v1.0/applications?user=brunor`

Please be aware that application installation API does not take care of FutureGateway defined uers groups and values. In order to allow users to run this application it is necessary to allow at least one group to run this application, inserting a new record in table fg_group_apps: insert into fg_group_apps values ( \<group_id\>, \<app_id\>, now());.

Inside `$FGLOCATION/fgAPIServer/`apps directory, there are several applications that can be used as template to generate new applications. Once selected the most appropriate application to use, just make a full copy of the application direcotry, provide to the new directory the application name and go inside of it. In the application directory there are two files: setup\_app.sh and stress\_test.sh. Modify the first file to setup properly the new application, use the second file to execute the new application on the system. The second script can be executed only after the setup\_app.sh script has been executed. The setup_app.sh script contains several configuration settings and it can be executed specifying as argument a session token (with no 'Authentication: Bearer' string).

Delete

`curl -i -H "Content-Type: application/json" -X DELETE`

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

## Debug with Liferay using Shibboleth
Once connected get from the browser shibboleth session and pass it as cookie; for instance:
`curl --cookie "_shibsession_64656661756c7468747470733a2f2f7367772e696e6469676f2d64617461636c6f75642e65752f73686962626f6c657468=_9dcb88c4f6b46005a1f2403079dd738f" -v https://sgw.indigo-datacloud.eu/apis/v1.0/tasks?user=brunor`

Another option is:

 1. Once logged place the section cookie in a 'cookie.txt' file
 2. In curl commands specify option: -b cookie.txt

## Infrastructures
Any installed application on the APIServer may have defined one or more infrastructures where it can execute. The current version of the software does not cover API specification on infrastructure management. At the moment it is only possible to create infrastructures when installing new applications where in the rest call instead to pass the infrastructure array of `<id>`s as reported in the specifications, it is necessary to pass an array of defined infrastructures. In this way there are no generic infrastructures to share among different applications but rather infrastructure defined at application level. This solution has been intentionally designed because by previous experiences in most of the cases each application requires its own specific settings. However the infrastructure sharing among application is still possible but only configuring properly the database directly.
For more information about, please have a look on the 'Advanced operation' section.


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

