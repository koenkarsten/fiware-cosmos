#<a name="top"></a>OAuth2-based authentication provider for Cosmos
* [What is cosmos-hive-auth-provider](#section1)
* [Installation](#section2)
    * [Prerequisites](#section2.1)
    * [Building](#section2.2)
    * [Jar copying](#section2.3)
    * [Unit tests](#section2.4)
* [Configuration](#section3)
* [Running](#section4)
* [Usage](#section5)
* [Administration](#section6)
* [Reporting issues and contact information](#section7)

##<a name="section1"></a>What is cosmos-hive-auth-provider
cosmos-hive-auth-provider is a custom authentication provider for [Hive](https://hive.apache.org/). Hive natively provides many ways of implementing authentication, e.g. [Kerberos, PAM or LDAP](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2#SettingUpHiveServer2-Authentication/SecurityConfiguration), but it also allows for configuring custom mechanisms, like this one.

By using cosmos-hive-auth-provider users will be able to authenticate by means or their [OAuth2](https://www.google.es/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0ahUKEwjOj8WZpszKAhXGtBoKHTCvBLMQFgggMAA&url=http%3A%2F%2Foauth.net%2F2%2F&usg=AFQjCNG58oSRksGnoIWIhfWYBB1sg_YGng&sig2=9Sux2Pq4TJwspuoLrFoFrQ) token, generated by a OAuth2 Tokens Generator (a third party) handled by any trusted Identity Manager (for instance, [FIWARE Lab one](https://account.lab.fiware.org/)).

The advantage regarding the way this library has been implemented is that any user-and-password-based Hive client will continue working; simply, the password configuration parameter takes the token value.

[Top](#top)

##<a name="section2"></a>Installation
###<a name="section2.1"></a>Prerequisites
[`git`](https://git-scm.com/) tool and [Maven](https://maven.apache.org/) must be installed in order to download and build the cosmos-hive-auth-provider library, respectively. It is not the goal of this document to provide detailed installation instructions about the mentioned tools.

Of course, this library has no sense if no [Hive](https://hive.apache.org/) service is deployed in a [Hadoop](http://hadoop.apache.org/) cluster.

[Top](#top)

###<a name="section2.2"></a>Building
Start by cloning the `fiware-cosmos` repository at Github:

    $ git clone https://github.com/telefonicaid/fiware-cosmos.git
   
That will create a `fiware-cosmos` folder. Then, the library is built by runing the following `mvn` command under `fiware-cosmos/cosmos-hive-auth-provider`:

```
$ cd fiware-cosmos/cosmos-hive-auth-provider
$ mvn clean compile assembly:single
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building cosmos-hive-auth-provider 0.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ cosmos-hive-auth-provider ---
[INFO] Deleting /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ cosmos-hive-auth-provider ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ cosmos-hive-auth-provider ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/target/classes
[INFO] 
[INFO] --- maven-assembly-plugin:2.2-beta-5:single (default-cli) @ cosmos-hive-auth-provider ---
...
...
[INFO] META-INF/MANIFEST.MF already added, skipping
[INFO] META-INF/ already added, skipping
[INFO] META-INF/MANIFEST.MF already added, skipping
[INFO] org/ already added, skipping
[INFO] org/json/ already added, skipping
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13.144 s
[INFO] Finished at: 2016-01-27T17:44:07+01:00
[INFO] Final Memory: 103M/1993M
[INFO] ------------------------------------------------------------------------
```

[Top](#top)

###<a name="section2.3"></a>Jar copying
The cosmos-hive-auth-provider jar containing the `OAuth2AuthenticationProviderImpl` class must be copied into one folder within the Hive classpath, for instance:

```
$ ls /usr/lib/hive/lib/ | grep cosmos
cosmos-hive-auth-provider-0.0.0-SNAPSHOT-jar-with-dependencies.jar
```

[Top](#top)

###<a name="section2.4"></a>Unit tests
The unit tests are run by invoking this parameterized `mvn test` command:

```
$ mvn test -Duser=frb -Dtoken=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building cosmos-hive-auth-provider 0.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ cosmos-hive-auth-provider ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ cosmos-hive-auth-provider ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ cosmos-hive-auth-provider ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ cosmos-hive-auth-provider ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ cosmos-hive-auth-provider ---
[INFO] Surefire report directory: /home/fiware-portal/fiware-cosmos/cosmos-hive-auth-provider/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.telefonica.iot.cosmos.hive.authprovider.OAuth2AuthenticationProviderImplTest
Testing OAuth2AuthenticationProviderImpl.Authenticate, it must fail because the token does not exist
log4j:WARN No appenders could be found for logger (com.telefonica.iot.cosmos.hive.authprovider.HttpClientFactory).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Testing OAuth2AuthenticationProviderImpl.Authenticate, it must success
Testing OAuth2AuthenticationProviderImpl.Authenticate, it must fail because the user and the token don't match
Tests run: 3, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.256 sec

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.110 s
[INFO] Finished at: 2016-01-28T10:59:10+01:00
[INFO] Final Memory: 23M/481M
[INFO] ------------------------------------------------------------------------
```

[Top](#top)

##<a name="section3"></a>Configuration
Nothing has to be configured regarding cosmos-hive-auth-provider itself, but the Hive service must be configured in order to use it.

The following properties must be added to `hive-site.xml` in order to enable a custom authentication provider, i.e. cosmos-hive-auth-provider and its `OAuth2AuthenticationProviderImpl` class:

```
<property>
   <name>hive.server2.authentication</name>
   <value>CUSTOM</value>
</property>

<property>
   <name>hive.server2.custom.authentication.class</name>
   <value>com.telefonica.iot.cosmos.hive.authprovider.OAuth2AuthenticationProviderImpl</value>
</property>
```

This other property must be added to `hive-site.xml` if we want to overwrite the default value for the Identity Manager endpoint (the one validating the OAuth2 authentication tokens):

```
<property>
   <name>com.telefonica.iot.idm.endpoint</name>
   <value>https://account.lab.fiware.org</value>
</property>
```

Finally, this property must be modified in order to enable impersonation (on the contrary, all the queries are executed by the user `hive` instead of the real end user):

```
<property>
   <name>hive.server2.enable.doAs</name>
   <value>true</value>
</property>
```

[Top](#top)

##<a name="section4"></a>Running
This is a library directly used by Hive. In order HiveServer2 knows about it, restart the service from your cluster manager (e.g. Ambari) or from the command line:

    $ (sudo) service hive-server2 restart

[Top](#top)

##<a name="section5"></a>Usage
The comsos-hive-auth-provider library is used when a HiveServer2 client connects and pass a user and a OAuth2 token. For instance, let's assume we are using any of Hive clients given with [Cygnus](https://github.com/telefonicaid/fiware-cygnus/tree/master/resources/hiveclients) tool:

```
$ pwd
/home/centos/fiware-cygnus/resources/hiveclients/java/hiveserver2-client
$ mvn exec:java -Dexec.args="computing.cosmos.lab.fiware.org 10000 default frb xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building hiveserver2-client 0.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] >>> exec-maven-plugin:1.2.1:java (default-cli) > validate @ hiveserver2-client >>>
[INFO] 
[INFO] <<< exec-maven-plugin:1.2.1:java (default-cli) < validate @ hiveserver2-client <<<
[INFO] 
[INFO] --- exec-maven-plugin:1.2.1:java (default-cli) @ hiveserver2-client ---
Connecting to jdbc:hive2://computing.cosmos.lab.fiware.org:10000/default?user=frb&password=XXXXXXXXXX
remotehive> show tables;
frb_test
remotehive> describe frb_test;
name,string
job,string
age,int
remotehive> select * from frb_test;
han,smuggler,35
luke,jedi,32
leia,princess,28
r2d2,robot,15
remotehive>
```

[Top](#top)

##<a name="section6"></a>Administration
HiveServer2 traces are usually logged within `/var/log/hive/hiveserver2.log`. There, you can find the traces regarding cosmos-hive-auth-provider, for instance, if everything goes well:

```
2016-01-27 15:59:17,587 INFO  [pool-5-thread-4]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(67)) - Doing request: GET https://account.lab.fiware.org/user?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx HTTP/1.1
2016-01-27 15:59:17,594 INFO  [pool-5-thread-4]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(78)) - Response received: {"organizations": [], "displayName": "frb", "roles": [{"name": "provider", "id": "106"}], "app_id": "8556cc76154f41b3b43d7b31f0699982", "email": "frb@tid.es", "id": "frb"}
2016-01-27 15:59:17,667 INFO  [pool-5-thread-4]: thrift.ThriftCLIService (ThriftCLIService.java:OpenSession(188)) - Client protocol version: HIVE_CLI_SERVICE_PROTOCOL_V6
2016-01-27 15:59:17,676 INFO  [pool-5-thread-4]: hive.metastore (HiveMetaStoreClient.java:open(297)) - Trying to connect to metastore with URI thrift://dev-fiwr-bignode-11.hi.inet:9083
2016-01-27 15:59:17,677 INFO  [pool-5-thread-4]: hive.metastore (HiveMetaStoreClient.java:open(385)) - Connected to metastore.
```

If the token does not exist, this is an example of relevant traces:

```
2016-01-27 16:10:10,196 INFO  [pool-5-thread-28]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(67)) - Doing request: GET https://account.lab.fiware.org/user?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx HTTP/1.1
2016-01-27 16:10:10,197 INFO  [pool-5-thread-28]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(78)) - Response received: {"error": {"message": "Access Token xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx not found", "code": 404, "title": "Not Found"}}
2016-01-27 16:10:10,197 ERROR [pool-5-thread-28]: transport.TSaslTransport (TSaslTransport.java:open(296)) - SASL negotiation failure
```

If the token exists but does not match the given user, then something like this is logged:

```
2016-01-27 16:12:11,520 INFO  [pool-5-thread-32]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(67)) - Doing request: GET https://account.lab.fiware.org/user?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx HTTP/1.1
2016-01-27 16:12:11,521 INFO  [pool-5-thread-32]: authprovider.HttpClientFactory (OAuth2AuthenticationProviderImpl.java:Authenticate(78)) - Response received: {"organizations": [], "displayName": "frb", "roles": [{"name": "provider", "id": "106"}], "app_id": "8556cc76154f41b3b43d7b31f0699982", "email": "frb@tid.es", "id": "frb"}
2016-01-27 16:12:11,521 ERROR [pool-5-thread-32]: transport.TSaslTransport (TSaslTransport.java:open(296)) - SASL negotiation failure
javax.security.sasl.SaslException: Error validating the login [Caused by javax.security.sasl.AuthenticationException: The given token does not match the given user]
```

[Top](#top)

##<a name="section7"></a>Reporting issues and contact information
There are several channels suited for reporting issues and asking for doubts in general. Each one depends on the nature of the question:

* Use [stackoverflow.com](http://stackoverflow.com) for specific questions about this software. Typically, these will be related to installation problems, errors and bugs. Development questions when forking the code are welcome as well. Use the `fiware-cosmos` tag.
* Use [ask.fiware.org](https://ask.fiware.org/questions/) for general questions about FIWARE, e.g. how many cities are using FIWARE, how can I join the accelarator program, etc. Even for general questions about this software, for instance, use cases or architectures you want to discuss.
* Personal email:
    * [francisco.romerobueno@telefonica.com](mailto:francisco.romerobueno@telefonica.com) **[Main contributor]**
    * [fermin.galanmarquez@telefonica.com](mailto:fermin.galanmarquez@telefonica.com) **[Contributor]**
    * [german.torodelvalle@telefonica.com](mailto:german.torodelvalle@telefonica.com) **[Contributor]**
    * [pablo.coellovillalba@telefonica.com](mailto:pablo.coellovillalba@telefonica.com) **[Contributor]**

**NOTE**: Please try to avoid personaly emailing the contributors unless they ask for it. In fact, if you send a private email you will probably receive an automatic response enforcing you to use [stackoverflow.com](stackoverflow.com) or [ask.fiware.org](https://ask.fiware.org/questions/). This is because using the mentioned methods will create a public database of knowledge that can be useful for future users; private email is just private and cannot be shared.

[Top](#top)