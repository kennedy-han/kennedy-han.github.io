---
layout: post
title: "Gsnova on Heroku"
description: "Gsnova on Heroku"
category: Tech
tags: [Gsnova, Heroku]
---

#Deploy Gsnova

`Aim`:
    Deploy a proxy application(Gsnova) on PaaS(Herokug) and use it

`Heroku` is a platform as a service (PaaS) that enables developers to build and run applications entirely in the cloud. (can create free account)

https://heroku.com

`Gsnova` inherit from `snova` , a Proxy application on PaaS

https://github.com/yinqiwen/gsnova

##step 1 Heroku Client env
register a account from Heroku and install `Heroku Toolbelt`

Windows can use CMD.exe or Cygwin

##Step 2 deploy Snova C4 server on Heroku
download snova-c4-server-[version].war

https://code.google.com/p/snova/downloads/list

```
cd $war's_directory

heroku login

heroku plugins:install https://github.com/heroku/heroku-deploy  --just run only once

heroku apps:create      --create a app，name in random ,please remember this appname ($app_name.herokuapp.com), It is not nessesary run this when update

heroku deploy:war --war <path_to_war_file> --app <app_name> 

heroku apps  --list all apps

Creating evening-basin-7128... done, stack is cedar-14
https://evening-basin-7128.herokuapp.com/ | https://git.heroku.com/evening-basin
-7128.git

heroku deploy:war --war snova-c4-java-server-0.22.0.war --app evening-basin-7128
```

In this example, my app name is `evening-basin-7128`

####troubleshooting
When you deploy the war, caught a SSL Exception:

```
D:\dev\snova-server>heroku deploy:war --war snova-c4-java-server-0.22.0.war --ap
p evening-basin-7128
Uploading snova-c4-java-server-0.22.0.war....
-----> Packaging application...
       - app: evening-basin-7128
       - including: webapp-runner.jar
       - including: snova-c4-java-server-0.22.0.war
-----> Creating build...
       - file: slug.tgz
       - size: 8MB
-----> Uploading build...
Exception in thread "main" javax.net.ssl.SSLHandshakeException: Remote host clos
ed connection during handshake
        at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:992)
        at sun.security.ssl.SSLSocketImpl.performInitialHandshake(SSLSocketImpl.
java:1375)
        at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1403
)
        at sun.security.ssl.SSLSocketImpl.startHandshake(SSLSocketImpl.java:1387
)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.createLayeredSock
et(SSLConnectionSocketFactory.java:395)
        at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSL
ConnectionSocketFactory.java:354)
        at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect
(DefaultHttpClientConnectionOperator.java:134)
        at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(
PoolingHttpClientConnectionManager.java:353)
        at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClie
ntExec.java:380)
        at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.
java:236)
        at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java
:184)
        at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:88)
        at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java
:110)
        at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttp
Client.java:184)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttp
Client.java:82)
        at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttp
Client.java:107)
        at com.heroku.sdk.deploy.utils.RestClient.put(RestClient.java:120)
        at com.heroku.sdk.deploy.endpoints.ApiEndpoint.upload(ApiEndpoint.java:4
0)
        at com.heroku.sdk.deploy.BuildsDeployer.deploySlug(BuildsDeployer.java:9
9)
        at com.heroku.sdk.deploy.Deployer.createAndReleaseSlug(Deployer.java:108
)
        at com.heroku.sdk.deploy.Deployer.deploy(Deployer.java:69)
        at com.heroku.sdk.deploy.App.deploy(App.java:57)
        at com.heroku.sdk.deploy.App.deploy(App.java:61)
        at com.heroku.sdk.deploy.WarApp.deploy(WarApp.java:30)
        at com.heroku.sdk.deploy.DeployWar.main(DeployWar.java:70)
Caused by: java.io.EOFException: SSL peer shut down incorrectly
        at sun.security.ssl.InputRecord.read(InputRecord.java:505)
        at sun.security.ssl.SSLSocketImpl.readRecord(SSLSocketImpl.java:973)
        ... 24 more
```

Solution: open `VPN` or set `$http_proxy` `$https_proxy` and re-try

```
D:\dev\snova-server>heroku deploy:war --war snova-c4-java-server-0.22.0.war --ap
p evening-basin-7128
Uploading snova-c4-java-server-0.22.0.war....
-----> Packaging application...
       - app: evening-basin-7128
       - including: webapp-runner.jar
       - including: snova-c4-java-server-0.22.0.war
-----> Creating build...
       - file: slug.tgz
       - size: 8MB
-----> Uploading build...
       - success
-----> Deploying...
remote:
remote: -----> Fetching set buildpack https://codon-buildpacks.s3.amazonaws.com/
buildpacks/heroku/jvm-common.tgz... done
remote: -----> JVM Common app detected
remote: -----> Installing OpenJDK 1.8... done
remote:
remote: -----> Discovering process types
remote:        Procfile declares types -> web
remote:
remote: -----> Compressing... done, 57.5MB
remote: -----> Launching... done, v3
remote:        https://evening-basin-7128.herokuapp.com/ deployed to Heroku
remote:
-----> Done
```

##step 3 install Client
https://code.google.com/p/snova/downloads/list

##step 4 Configure the Client
edit Client config file, replace the app name

vi /gsnova-0.22.1/gsnova.conf
```
[LocalServer]
Listen=0.0.0.0:48100
[C4]
Enable=1
Listen=0.0.0.0:48102
WorkerNode[0]=https://evening-basin-7128.herokuapp.com
ReadTimeout = 25
MaxConn = 3
WSConnKeepAlive = 1800
Compressor=Snappy
Encrypter=RC4
UseSysDNS=0
MultiRangeFetchEnable=0
RangeFetchLimitSize=262144
RangeConcurrentFetcher=3
InjectRange=*.c.youtube.com|av.vimeo.com|av.voanews.com
UserAgent=Mozilla/5.0 (Windows NT 6.1; WOW64; rv:15.0) Gecko/20100101 Firefox/15.0.1
Proxy=
```

##step 5 use proxy

once configurationed, Run client app:

```
./gsnova
```

I use Chrome with `SwitchySharp` plugin

or set `$http_proxy` to use the client proxy ip

enjoy it

##Complie Gsnova
read README file on
https://github.com/yinqiwen/gsnova

###troubleshooting
When compiling error, try remove `icon.syso` file

#Reference
https://github.com/yinqiwen/gsnova/blob/develop/README.md