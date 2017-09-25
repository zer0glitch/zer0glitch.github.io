---
layout: post
section-type: post
category: life
tags: [ 'jboss,haveged' ]
title: "Jboss Hanging on Start-up"
description: Jboss Hanging on Start-up
---

**Jboss Timeout on startup**

*Issue*

```
[org.jboss.as.controller.management-operation] (Controller Boot Thread) JBAS013412: Timeout after [300] seconds waiting for service container stability. Operation will roll back. Step that first updated the service container was 'add' at address '[("interface" => "management")]'
```

*Debugging*

To debug this issue you can use jstack and do a thread dump.  Ensure that you are the user that is running the jboss process.

1.  Find the process ID of the user running JBOSS

```
ps -ef | grep jboss
root      1093     1  0 Sep22 ?        00:00:00 su - jboss -c cd /opt/jboss/jboss-current; LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=/var/run/jboss-eap/jboss-eap-standalone.pid /opt/jboss/jboss-current/bin/standalone.sh -c standalone.xml
jboss     2274  1093  0 Sep22 ?        00:00:00 -bash -c cd /opt/jboss/jboss-current; LAUNCH_JBOSS_IN_BACKGROUND=1 JBOSS_PIDFILE=/var/run/jboss-eap/jboss-eap-standalone.pid /opt/jboss/jboss-current/bin/standalone.sh -c standalone.xml
jboss     2370  2274  0 Sep22 ?        00:00:00 /bin/sh /opt/jboss/jboss-current/bin/standalone.sh -c standalone.xml
jboss     2570  2370  0 Sep22 ?        00:28:01 /etc/alternatives/java_sdk/bin/java -D[Standalone] -server -XX:+UseCompressedOops -verbose:gc -Xloggc:/opt/jboss/jboss-eap-6.4/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms1895M -Xmx1895M -Djava.net.preferIPv4Stack=true -Dorg.jboss.resolver.warning=true -Dsun.rmi.dgc.client.gcInterval=3600000 -Dsun.rmi.dgc.server.gcInterval=3600000 -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -Djboss.server.default.config=standalone.xml -Djavax.net.ssl.keyStore=/etc/pki/java/keystore.jks -Djavax.net.ssl.keyStorePassword=changeit -Djavax.net.ssl.trustStore=/etc/pki/java/truststore.jks -Djavax.net.ssl.trustStorePassword=changeit -Dorg.jboss.boot.log.file=/opt/jboss/jboss-eap-6.4/standalone/log/server.log -Dlogging.configuration=file:/opt/jboss/jboss-eap-6.4/standalone/configuration/logging.properties -jar /opt/jboss/jboss-eap-6.4/jboss-modules.jar -mp /opt/jboss/jboss-eap-6.4/modules -jaxpmodule javax.xml.jaxp-provider org.jboss.as.standalone -Djboss.home.dir=/opt/jboss/jboss-eap-6.4 -Djboss.server.base.dir=/opt/jboss/jboss-eap-6.4/standalone -c standalone.xml
root     13879 13754  0 11:32 pts/0    00:00:00 su - jboss
jboss    13880 13879  0 11:32 pts/0    00:00:00 -bash
jboss    13936 13880  0 11:32 pts/0    00:00:00 ps -ef
jboss    13937 13880  0 11:32 pts/0    00:00:00 grep --color=auto jboss
```

2.  Peform the thread tump

````
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-2.b16.el7_4.x86_64/bin/jstack -l 2570 > jstack.out
```

3.  Analyze the thread and look for wating and locked processes


*Solutions*

1.  Point /dev/random at /dev/urandom.  This is a less secure method of generating random numbers
```
rm -rf /dev/random
ln -s /dev/urandom /dev/random
```

2.  Increase the timeout for the applicaiton deployment
- https://access.redhat.com/solutions/1190323

3.  Install haveged
```
yum install -y haveged
```
