---
layout: post
title: "Automated Testing with Ansible"
description: Utilizing Selenium and PhantomJS as an Ansible Module for configuration or testing
---

Ansible is farily flexable and extensible when for automation tool that can be extended through the use of custom modules.  This post will walk through the steps of using Selenium Web Driver scripts generated through the Mozilla plugin, to modify that into an ansible module, to do a web based configuration. 

This project will have all code available in the example repoistory on github.  Here are the requirements for getting started:

Initial Setup
====

**Downloads**

* [Installing Ansible](http://docs.ansible.com/ansible/intro_installation.html)
* [Installing Selenium Firefox Plugin](https://addons.mozilla.org/en-US/firefox/addon/selenium-ide/)
* [Installing PhantomJS](http://phantomjs.org/download.html)


- Install EPEL Repo

```
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
```

- Install the support files

```
yum install sshpass python-pip pycrypto -y
pip install ansible
```

- Selenium Install

```
pip install selenium
```

- Installing PhantomJS

```
wget -P /tmp https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2
cd /tmp
bunzip2 phantomjs-2.1.1-linux-x86_64.tar.bz2
tar xvf phantomjs-2.1.1-linux-x86_64.tar
cp phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/local/bin
chmod 755 /usr/local/bin/phantomjs
```

Generating your Selenium Script
====

- Open Firefox and Launch the Selenium IDE
- Click the record button
- goto: http://localhost/samtest/index.jsp
- Enter any value in the name field
- Click submit
- Goto file --> Export --> python2 web driver

The defautl script will be setup to run an open instance of Firefox.  Later when we modify the script will change it to use PhantomJS, so this can be integrated into a CICD process.


- 
