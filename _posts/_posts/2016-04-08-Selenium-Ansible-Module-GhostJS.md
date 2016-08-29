---
layout: post
title: "Using Selenium and Ghostdriver as an Ansible Module"
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


Exam the saved script and convert to an ansible module
====
1. Create a directory called library under your base playbooks
2. Copy the generated script to the directory
3. Make modifications into an ansible module

- We add the path to 


```
#!/usr/bin/python

# -*- coding: utf-8 -*-
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import Select
from selenium.common.exceptions import NoSuchElementException
from selenium.common.exceptions import NoAlertPresentException
import unittest, time, re

```

- Add Ansible Inport modules


```
# Add Ansible Inport modules
from ansible.module_utils.basic import *

```

- Change the Example module from a unit test a standard class

```
#class ExampleModule(unittest.TestCase):
class ExampleModule():

```

- Change the init to add the parameters we need and intiialize object variables

```
    def __init__(self, server, cert, certkey, certpass, creates):
        self.server = server
        self.creates = creates
        self.cert = cert
        self.certkey = certkey
        self.certpass = certpass

```
- make the setup of our driver part of the init for the object

```
#    def setUp(self):

```

- Comment out the web driver for firefox and add the driver for PhantomJS

```
        #self.driver = webdriver.Firefox()
        self.driver = webdriver.PhantomJS(executable_path="/usr/local/bin/phantomjs")
```

- The arguments below will allow you to connect to PKI proected or self generated certificate server

```
		#,service_args=['--ignore-ssl-errors=yes',
                #'--ssl-client-certificate-file=' + self.cert, 
                #'--ssl-client-key-file=' + self.certkey,
                #'--ssl-client-key-passphrase=' + self.certpass])

        self.driver.set_window_size(1120, 550)
        self.driver.implicitly_wait(30)
        self.base_url = "https://" + self.server
        self.verificationErrors = []
        self.accept_next_alert = True
    
    def config_service(self):
        changed = True
        success = True
        errors = []
        changes = []
```

- We add logic to not run if the file we have created already exists

```
        if not os.path.exists(self.creates):
            try:
```
- The section below is going to be used for the actual interaction, you need to replace specifics with generics where necessary

```
                driver = self.driver
                driver.get(self.base_url + "/example_module/web/")
                driver.find_element_by_id("password").clear()
                driver.find_element_by_id("password").send_keys("example_module")
                driver.find_element_by_id("username").clear()
                driver.find_element_by_id("username").send_keys("admin")
                driver.find_element_by_xpath("//button[@type='submit']").click()
                driver.find_element_by_link_text("Add stores").click()
                #driver.find_element_by_css_selector("#id5 > ul > li > div > a > span").click()
                driver.find_element_by_xpath("//form/ul/li/div/a/span").click()
                driver.find_element_by_name("parametersPanel:parameters:2:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:2:parameterPanel:border:paramValue").send_keys("admin")
                driver.find_element_by_name("parametersPanel:parameters:3:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:3:parameterPanel:border:paramValue").send_keys("example_module")
                driver.find_element_by_name("dataStoreNamePanel:border:paramValue").clear()
                driver.find_element_by_name("dataStoreNamePanel:border:paramValue").send_keys("tcri_ais_so")
                driver.find_element_by_name("parametersPanel:parameters:0:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:0:parameterPanel:border:paramValue").send_keys("tcri")
                driver.find_element_by_name("parametersPanel:parameters:1:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:1:parameterPanel:border:paramValue").send_keys("zoo1,zoo2,zoo3")
                driver.find_element_by_name("parametersPanel:parameters:2:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:2:parameterPanel:border:paramValue").send_keys("tcri_rw")
                driver.find_element_by_name("parametersPanel:parameters:3:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:3:parameterPanel:border:paramValue").send_keys("spawarinstall")
                driver.find_element_by_name("parametersPanel:parameters:4:parameterPanel:border:paramValue").clear()
                driver.find_element_by_name("parametersPanel:parameters:4:parameterPanel:border:paramValue").send_keys("geomesa")
                #driver.find_element_by_id("id9").click()
                driver.find_element_by_link_text("Save").click()
                driver.find_element_by_xpath("//span/a/span").click()
                driver.find_element_by_id("minX").clear()
                driver.find_element_by_id("minX").send_keys("-180")
                driver.find_element_by_id("minY").clear()
                driver.find_element_by_id("minY").send_keys("-90")
                driver.find_element_by_id("maxX").clear()
                driver.find_element_by_id("maxX").send_keys("180")
                driver.find_element_by_id("maxY").clear()
                driver.find_element_by_id("maxY").send_keys("90")
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:minX").clear()
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:minX").send_keys("-180")
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:minY").clear()
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:minY").send_keys("-90")
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:maxX").clear()
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:maxX").send_keys("180")
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:maxY").clear()
                driver.find_element_by_name("tabs:panel:theList:0:content:referencingForm:latLonBoundingBox:maxY").send_keys("90")
                driver.find_element_by_link_text("Save").click()
```

- Add a changed message here.  It is possible to add mutliple messages to display back in the ansible script

```
                changes.append("Added ais_oth_so layer")
```

- We will catch exceptions here and add error messages

```
            except Exception as e:
		# Set success to false for ansible notification
                success = False
		# Set changed to false for ansible notification
                changed = False
		# append the error stack
                errors.append("ERROR: ------------ " + str(e))
```

- if there are no errors create the creates dir

```
        if len(errors) == 0 and len(self.creates) != 0:
            if os.path.exists(self.creates):
                os.utime(self.creates, None)
            else:
                open(self.creates, 'a').close()
```

- return back a new dictionary, ansible will use this as part of its return

```
        return dict(success=success, changed=changed, changes=changes, errors=errors, msg=", ".join(errors))
```

- It is always import to close down the phatomjs browser, just because you don't see it, doesn't meant it isn't running
        
```
    def tearDown(self):
        self.driver.quit()
```

- Create our own main method

```
def main():
```
- this is commented out for testing purporses.  If you want to test outside of ansible then uncomment this ling and comemnt the rest of the main method

```
    #example_module = ExampleModule("172.26.14.35:8443", "./c3test_u_fouo.crt.pem", "./c3test_u_fouo.key.pem", "pass", "/tmp/geosotest")
```

- initialize the return value, by default we are goign to say everythign worked ok

```
    returnVal = dict(succes=True)
```

- We are defining each of the arguments that will be passed from the playbook here

```
    BASE_MODULE_ARGS = dict(
        server = dict(required=True),
        creates = dict(default=""),
        cert = dict(required=True),
        certkey = dict(required=True),
        certpass = dict(required=True)
    )
```
- Define the module we will be calling here

```
    module = AnsibleModule(
        argument_spec= BASE_MODULE_ARGS,
        supports_check_mode=True
    )
```

- We are now creating an instance of the python object, using the parameters that are pulled from above

```
    example_module = ExampleModule(
        module.params["server"],
        module.params["cert"],
        module.params["certkey"],
        module.params["certpass"],
        module.params["creates"]
    )
```
    
- try to call our test method, you can also add if statements based upon parameters here

```

    try:
        returnVal = example_module.test_geo_config()
```
- if there is an exception in the method exit the module with the error

```
    except Exception as e:
        module.fail_json(msg=str(e))
```
- make sure the browser is closed

```
    finally:
        example_module.tearDown()

```
- return with success if everything changed ok, otherwise exit with the error message

```
    if (returnVal['changed'] == True):
        module.exit_json(**returnVal)
    else:
        module.fail_json(msg=returnVal['msg'])
```
- try to call our test method, you can also add if statements based upon parameters here

```
# this is magic, see lib/ansible/module_common.py
# #<<INCLUDE_ANSIBLE_MODULE_COMMON>>
if __name__ == "__main__":
    main()
```

- comment out the unit test that would have been used for testing

```
#    unittest.main()
```


Actually Using selenium and python for testing
===

In another article we will talk about automated testing instead of configuration with the same tool set


Download full example
===
[http://zer0glitch.githbu.com/zer0glitch/](http://zer0glitch.githbu.com/zer0glitch/)
