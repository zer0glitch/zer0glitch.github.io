---
layout: post
section-type: post
category: tech
tags: [ 'tutorial' ]
title: "Create an RPM out of an archive using maven"
description: Creating an RPM for wildfly using maven.
---

***Introduction***


On Linux based systems, it is best to use a package manager to install, update, and delete software on your system.  This allows for software to be installed in a common way.  User, Groups, Permissions, and location will be consistent across your environment.  

**Tools**

* Maven
* maven-rpm-plugin
* maven-download-plugin
* buildtools

The sample pom below does all the work.

If you are building for another project you need to change:

* artifactId
* groupId
* version
* download url

By default this will install in /opt.  If you would like to deploy somewhere else change the output location, permissions, group and  username, if you destire.

**Sample Pom**

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>wildfly</groupId>
  <artifactId>wildfly</artifactId>
  <version>10.1.0.CR1</version>
  <packaging>pom</packaging>
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <wildfly.url/>
  </properties>
  <build>
    <plugins>
      <plugin>
        <groupId>com.googlecode.maven-download-plugin</groupId>
        <artifactId>download-maven-plugin</artifactId>
        <version>1.3.0</version>
        <executions>
          <execution>
            <!-- the wget goal actually binds itself to this phase by default -->
            <phase>process-resources</phase>
            <goals>
              <goal>wget</goal>
            </goals>
            <configuration>
              <url>http://download.jboss.org/wildfly/10.1.0.CR1/wildfly-10.1.0.CR1.zip</url>
              <outputFileName>${project.artifactId}-${project.version}.zip</outputFileName>
              <!-- default target location, just to demonstrate the parameter -->
              <targetDirectory>${project.build.directory}</targetDirectory>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>truezip-maven-plugin</artifactId>
        <version>1.2</version>
        <executions>
          <execution>
            <id>copy-package</id>
            <goals>
              <goal>copy</goal>
            </goals>
            <phase>process-resources</phase>
            <configuration>
              <verbose>true</verbose>
              <fileset>
                <directory>target/${project.artifactId}-${project.version}.zip</directory>
                <outputDirectory>target</outputDirectory>
              </fileset>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>rpm-maven-plugin</artifactId>
        <executions>
          <execution>
            <id>generate-rpm</id>
            <goals>
              <goal>attached-rpm</goal>
            </goals>
            <phase>package</phase>
          </execution>
        </executions>
        <configuration>
          <name>${project.artifactId}</name>
          <group>Wildfly/Deps</group>
          <copyright>GPL, v2</copyright>
          <prefix>/opt/${project.artifactId}-${project-version}</prefix>
          <needarch>noarch</needarch>
          <targetOS>linux</targetOS>
          <defaultDirmode>755</defaultDirmode>
          <defaultFilemode>755</defaultFilemode>
          <defaultUsername>root</defaultUsername>
          <defaultGroupname>root</defaultGroupname>
          <defineStatements>
            <defineStatement>_unpackaged_files_terminate_build 0</defineStatement>
            <defineStatement>__jar_repack 0</defineStatement>
          </defineStatements>
          <directoryIncluded>true</directoryIncluded>
          <mappings>
            <mapping>
              <directory>/opt/${project.artifactId}-${project.version}</directory>
              <username>root</username>
              <groupname>root</groupname>
              <sources>
                <source>
                  <location>target/${project.artifactId}-${project.version}</location>
                </source>
              </sources>
            </mapping>
          </mappings>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
<!-- vim: set ts=4 sw=4: -->

```


**Run maven**

```bash
mvn clean install
```

The output is a wildfly rpm that install in /opt/wildfly-10.1.0.CR1
