#<a name="top"></a>Tidoop REST API

* [What is cosmos-tidoop-api](#section1)
* [Installation](#section2)
    * [Prerequisites](#section2.1)
    * [API installation](#section2.2)
    * [Unit tests](#section2.3)
* [Configuration](#section3)
* [Running](#section4)
* [Usage](#section5)
* [Administration](#section6)
    * [Logging traces](#section6.1)
    * [Submitted jobs](#section6.2)
* [Contact](#section7)

##<a name="section1"></a>What is cosmos-tidoop-api
cosmos-tidoop-api exposes a RESTful API for running MapReduce jobs in a shared Hadoop environment.

Why emphasize in <i>a shared Hadoop environment</i>? Because shared Hadoops require special management of the data and the analysis processes being run (storage and computation). There are tools like [Oozie](https://oozie.apache.org/) in charge of running MapReduce jobs as well through an API, but they do not take into account the access to the run jobs, their status, results, etc must be controlled. In other words, using Oozie any user may kill a job by knowing its ID; using cosmos-tidoop-api only the owner of the job will be able to.

The key point is to relate all the MapReduce operations (run, kill, retrieve status, etc) to the user space in HDFS. This way, simple but effective authorization policies can be stablished per user space (in the most basic approach, allowing only a user to access it own user space). This can be easily combined with authentication mechanisms such as [OAuth2](http://oauth.net/2/).

Finally, it is important to remark cosmos-tidoop is being designed to run in a computing cluster, but in charge of analyzing the data within a storage cluster. Sometimes, of course, both storage and computing cluster may be the same, but they could be splited and this software is ready for that.

[Top](#top)

##<a name="section2"></a>Installation
This is a software written in JavaScript, specifically suited for [Node.js](https://nodejs.org) (<i>JavaScript on the server side</i>). JavaScript is an interpreted programming language thus it is not necessary to compile it nor build any package; having the source code downloaded somewhere in your machine is enough.

###<a name="section2.1"></a>Prerequisites
This REST API has no sense if tidoop-mr-lib is not installed. And tidoop-mr-lib has only sense in a [Hadoop](http://hadoop.apache.org/) cluster, thus both the library and Hadoop are required.

As said, cosmos-tidoop-api is a Node.js application, therefore install it from the official [download](https://nodejs.org/download/). An advanced alternative is to install [Node Version Manager](https://github.com/creationix/nvm) (nvm) by creationix/Tim Caswell, which will allow you to have several versions of Node.js and switch among them.

Of course, common tools such as `git` and `curl` are needed.

[Top](#top)

###<a name="section2.2"></a>API installation
Start by creating, if not yet created, a Unix user named `cosmos-tidoop`; it is needed for installing and running the application. You can only do this as root, or as another sudoer user:

    $ sudo useradd cosmos-tidoop
    $ sudo passwd cosmos-tidoop <choose_a_password>
    
While you are a sudoer user, create a folder for saving the cosmos-tidoop-api log traces under a path of your choice, typically `/var/log/cosmos/cosmos-tidoop-api`, and set `cosmos-tidoop` as the owner:

    $ sudo mkdir -p /var/log/cosmos/cosmos-tidoop-api
    $ sudo chown -R cosmos-tidoop:cosmos-tidoop /var/log/cosmos
    
Now it is time to enable the `cosmos-tidoop` user to run Hadoop commands as the requesting user. This can be done in two ways:

* Adding the `cosmos-tidoop` user to the sudoers list. This is the easiest way, but the most dangerous one.
* Adding the `cosmos-tidoop` user to all the user groups (by default, for any user there exists a group with the same name than the user). This is only useful if, and only if, the group permissions are as wide open as the user ones (i.e. `77X`).

Once , change to the new fresh `cosmos-tidoop` user:

    $ su - cosmos-tidoop

Then, clone the fiware-cosmos repository somewhere of your ownership:

    $ git clone https://github.com/telefonicaid/fiware-cosmos.git
    
cosmos-tidoop-api code is located at `fiware-cosmos/cosmos-tidoop-api`. Change to that directory and execute the installation command:

    $ cd fiware-cosmos/cosmos-tidoop-api
    $ npm install
    
That must download all the dependencies under a `node_modules` directory.

[Top](#top)

###<a name="section2.3"></a>Unit tests
To be done.

[Top](#top)

##<a name="section3"></a>Configuration
cosmos-tidoop-api is configured through a JSON file (`conf/cosmos-tidoop-api.json`). These are the available parameters:

* **host**: FQDN or IP address of the host running the service. Do not use `localhost` unless you want only local clients may access the service.
* **port**: TCP listening port for incomming API methods invocation. 12000 by default.
* **storage_cluster**:
    * **namenode_host**: FQDN or IP address of the Namenode of the storage cluster.
    * **namenode_ipc_port**: TCP listening port for Inter-Process Communication used by the Namenode of the storage cluster. 8020 by default.
* **log**:
    * **file_name**: Path of the file where the log traces will be saved in a daily rotation basis. This file must be within the logging folder owned by the the user `tidoop`.
    * **date_pattern**: Data pattern to be appended to the log file name when the log file is rotated.

[Top](#top)

##<a name="section4"></a>Running
The Http server implemented by cosmos-tidoop-api is run as (assuming your current directory is `fiware-cosmos/cosmos-tidoop-api`):

    $ npm start
    
If everything goes well, you should be able to remotely ask (using a web browser or `curl` tool) for the version of the software:

    $ curl -X GET "http://<host_running_the_api>:12000/tidoop/v1/version"
    {"version": "0.1.1"}
    
cosmos-tidoop-api typically listens in the TCP/12000 port, but you can change if by editing `conf/cosmos-tidoop-api.conf` as seen above.

[Top](#top)

##<a name="section5"></a>Usage
NOTE: A `X-Auth-Token` header has been included in all the requests assuming the API is protected by means of some kind of token-based authentication mechanism, such as [OAUth2](http://oauth.net/2/).

###<a name="section5.1"></a>`GET /tidoop/v1/version`
Gets the running version of cosmos-tidoop.

Request example:

```
GET http://<tidoop_host>:<tidoop_port>/tidoop/v1/version HTTP/1.1
X-Auth-Token: 3bzH35FFLdapMgVCOdpot23534fa8a
```

Response example:

```
HTTP/1.1 200 OK

{
    "success": "true", 
    "version": "0.2.0-next"
}
```

[Top](#top)

###<a name="section5.2"></a>`POST /tidoop/v1/user/{userId}/jobs`
Runs a MapReduce job given the following parameters:

* Java jar containing the desired MapReduce application.
* The name of the MapReduce application.
* Any additional library jars required by the application.
* The input HDFS directory in the storage cluster.
* The output HDFS directory in the storeage cluster.

Request example:

```
POST http://computing.cosmos.lab.fiware.org:12000/tidoop/v1/user/frb/jobs HTTP/1.1
Content-Type: application/json
X-Auth-Token: 3bzH35FFLdapMgVCOdpot23534fa8a

{
	"jar": "/usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar",
	"class_name": "wordcount",
	"lib_jars": "/usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar",
	"input": "mrtest",
	"output": "output4"
}
```

Response example:

```
HTTP/1.1 200 OK

{
    "success": "true",
    "job_id": "job_1460639183882_0005"
}
```

[Top](#top)

###<a name="section5.3"></a>`GET /tidoop/v1/user/{userId}/jobs`
Gets the details for all the MapReduce jobs run by the given user ID.

Request example:

```
GET http://computing.cosmos.lab.fiware.org:12000/tidoop/v1/user/frb/jobs HTTP/1.1
X-Auth-Token: 3bzH35FFLdapMgVCOdpot23534fa8a
```

Response example:

```
HTTP/1.1 200 OK

{
	"success": "true",
	"jobs": [{
		"job_id": "job_1460639183882_0005",
		"state": "SUCCEEDED",
		"start_time": "1460963556383",
		"user_id": "frb"
	}, {
		"job_id": "job_1460639183882_0004",
		"state": "SUCCEEDED",
		"start_time": "1460959583838",
		"user_id": "frb"
	}]
}
```

[Top](#top)

###<a name="section5.4"></a>`GET /tidoop/v1/user/{userId}/jobs/{jobId}`
Gets the details for the given MapReduce job run by the given user ID.

Request example:

```
GET http://computing.cosmos.lab.fiware.org:12000/tidoop/v1/user/frb/jobs/job_1460639183882_0005 HTTP/1.1
X-Auth-Token: 3bzH35FFLdapMgVCOdpot23534fa8a
```

Response example:

```
HTTP/1.1 200 OK

{
	"success": "true",
	"job": {
		"job_id": "job_1460639183882_0005",
		"state": "SUCCEEDED",
		"start_time": "1460963556383",
		"user_id": "frb"
	}
}
```

[Top](#top)

###<a name="section5.5"></a>`DELETE /tidoop/v1/user/{userId}/jobs/{jobId}`
Deletes the given MapReduce job run by the given user ID.

Request example:

```
DELETE http://computing.cosmos.lab.fiware.org:12000/tidoop/v1/user/frb/jobs/job_1460639183882_0005 HTTP/1.1
X-Auth-Token: 3bzH35FFLdapMgVCOdpot23534fa8a
```

Response example:

```
HTTP/1.1 200 OK

{
    "success": "true"
}
```

[Top](#top)

##<a name="section6"></a>Administration
Two are the sources of data for administration purposes, the logs and the list of jobs launched.

[Top](#top)

###<a name="section6.1"></a>Logging traces
Logging traces, typically saved under `/var/log/cosmos/cosmos-tidoop-lib`, are the main source of information regarding the GUI performance. These traces are written in JSON format, having the following fields: level, message and timestamp. For instance:

    {"level":"info","message":"Connected to http://130.206.81.225:3306/cosmos_gui","timestamp":"2015-07-31T08:44:04.624Z"}

Logging levels follow this hierarchy:

    debug < info < warn < error < fatal
    
Within the log it is expected to find many `info` messages, and a few of `warn` or `error` types. Of special interest are the errors:

* ***Some error occurred during the starting of the Hapi server***: This message may appear when starting the API. Most probably the configured host IP address/FQDN does not belongs to the physical machine the service is running, or the configured port is already used.

[Top](#top)

###<a name="section6.2"></a>Submitted jobs
As an administrator, information regarding submitted jobs can be retrieved via the `hadoop job` command (it must be said such a command is the underlying mechanism the REST API uses in order to return information regarding MapReduce jobs). A complete reference for this command can be found in the official [Hadoop documentation](https://hadoop.apache.org/docs/r1.2.1/commands_manual.html#job). 

[Top](#top)

##<a name="section7"></a>Reporting issues and contact information
There are several channels suited for reporting issues and asking for doubts in general. Each one depends on the nature of the question:

* Use [stackoverflow.com](http://stackoverflow.com) for specific questions about this software. Typically, these will be related to installation problems, errors and bugs. Development questions when forking the code are welcome as well. Use the `fiware-cosmos` tag.
* Use [ask.fiware.org](https://ask.fiware.org/questions/) for general questions about FIWARE, e.g. how many cities are using FIWARE, how can I join the accelarator program, etc. Even for general questions about this software, for instance, use cases or architectures you want to discuss.
* Personal email:
    * [francisco.romerobueno@telefonica.com](mailto:francisco.romerobueno@telefonica.com) **[Main contributor]**
    * [german.torodelvalle@telefonica.com](mailto:german.torodelvalle@telefonica.com) **[Contributor]**
    * [pablo.coellovillalba@telefonica.com](mailto:pablo.coellovillalba@telefonica.com) **[Contributor]**

**NOTE**: Please try to avoid personaly emailing the contributors unless they ask for it. In fact, if you send a private email you will probably receive an automatic response enforcing you to use [stackoverflow.com](stackoverflow.com) or [ask.fiware.org](https://ask.fiware.org/questions/). This is because using the mentioned methods will create a public database of knowledge that can be useful for future users; private email is just private and cannot be shared.

[Top](#top)
